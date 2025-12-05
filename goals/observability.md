# Unified Observability Platform - Implementation Plan

**Last Updated:** 2025-12-03

## Implementation Status

### ✅ Implemented

- **Logs with Embeddings:** `@h-j-petroleum/toolkit-logger` with TimescaleTransport writes logs to `application_logs` table with embeddings generated via local `embeddings-gpu` service (OpenAI-compatible HTTP API at `http://embeddings-1:8000`). Implemented and validated across hjps-ops services (approval-api, pdf-generator, timesheet-api, api-gateway).
- **Traces:** `@h-j-petroleum/toolkit-telemetry` captures OpenTelemetry traces and sends them to the observability stack.
- **Metrics (Prometheus):** `@h-j-petroleum/toolkit-metrics` exposes `/metrics` endpoints in services. Metrics are scraped by Prometheus.

### ⏳ Planned / Partially Implemented

- **Metrics → Timescale:** Schema exists (`metrics_samples` table), but `TimescaleMetricsExporter` is not yet implemented. Current metrics flow is via Prometheus scraping only. Timescale metrics ingestion will be layered on top later.
- **MCP Tools:** Designs and partial implementations exist for `search_errors`, `compare_test_runs`, `service_health`, but not yet fully rolled out to production.

### 📚 Implementation References

- **SQL Schemas:** Implemented in `hjps-ops/observability/init-db/`
- **CI Observability Reporter:** `hjps-ops/scripts/ci-log-reporter.mjs` matches the `test_runs`/`test_failures`/`lint_failures` sections
- **Toolkit Packages:** Published to GitHub Packages under `@h-j-petroleum` scope

## Vision

Create a **comprehensive observability platform** that unifies all development activity data—tests, logs, metrics, traces, and builds—in a single queryable system accessible to both humans (via dashboards) and agents (via APIs and semantic search).

### Scope Expansion

**From:** Local/CI test comparison tool  
**To:** **Agentic ERP Nervous System**

**The "Nervous System" Concept:**
Just as a nervous system transmits signals between the brain and body, this platform connects your **Agents (Brain)** with your **ERP Services (Body)**.

- **Senses (Observability):** Agents can "see" system state via logs, metrics, and traces.
- **Memory (Vector Store):** Agents can recall past errors, similar test runs, and documentation via `vector-toolkit`.
- **Reflexes (Automated Actions):** Agents can trigger self-healing or investigation workflows.

**Repository Integration:**
- **Infrastructure Host:** `hjps-ops` (Docker Compose stack)
- **AI/Vector Layer:** `vector-toolkit` (PostgresStore + Embeddings)
- **UI Layer:** `hjps-frontend` (OTEL instrumentation)
- **Shared Utils:** `hjps-toolkit` (Logger, Metrics, Telemetry)

---

## Architecture Overview

```mermaid
graph TB
    subgraph "Agentic ERP Ecosystem"
        direction TB
        
        subgraph "hjps-frontend (UI)"
            UI[Next.js App]
            OTEL[OTEL Instrumentation]
        end
        
        subgraph "hjps-ops (Services)"
            API1[Timesheet API]
            API2[Approval API]
            Logger[Toolkit Logger]
        end
        
        subgraph "Observability Stack (Docker)"
            TSDB[(TimescaleDB + pgvector)]
            Grafana[Grafana Dashboards]
            Collector[OTEL Collector]
            EmbedService[Local LLM Service<br/>(Embeddings)]
            CodingService[Coding Agent Service<br/>(vLLM)]
        end
        
        subgraph "Agent Layer"
            Agent[Gemini/Claude Agent]
            MCP[Observability MCP Server]
            VectorTK[Vector Toolkit]
        end
    end
    
    UI --> API1
    UI --> API2
    
    OTEL --> Collector
    Logger --> TSDB
    
    API1 --> Logger
    API2 --> Logger
    
    Collector --> TSDB
    
    TSDB --> Grafana
    TSDB --> MCP
    
    Agent --> MCP
    MCP --> VectorTK
    VectorTK --> TSDB
    VectorTK --> EmbedService
```

---

## Unified Database Schema

### Extensions Setup

```sql
-- Enable required PostgreSQL extensions
CREATE EXTENSION IF NOT EXISTS timescaledb;
CREATE EXTENSION IF NOT EXISTS vector;  -- pgvector for embeddings
CREATE EXTENSION IF NOT EXISTS pg_trgm;  -- Text search
```

### 1. Test Results & Self-Healing Tracking

```sql
-- Test runs (local vs CI comparison)
CREATE TABLE test_runs (
  timestamp TIMESTAMPTZ NOT NULL,
  run_id VARCHAR(255) NOT NULL,

  -- Source
  repository VARCHAR(255) NOT NULL,
  branch VARCHAR(255) NOT NULL,
  commit_sha VARCHAR(40) NOT NULL,
  source VARCHAR(20) NOT NULL,  -- 'local' or 'ci'

  -- Environment
  node_version VARCHAR(20),
  pnpm_version VARCHAR(20),
  lockfile_hash VARCHAR(64),

  -- Results
  total_tests INT NOT NULL,
  passed_tests INT NOT NULL,
  failed_tests INT NOT NULL,
  skipped_tests INT NOT NULL,
  duration_ms INT NOT NULL,

  -- Self-healing context
  triggered_selfheal BOOLEAN DEFAULT false,
  selfheal_proposal_id VARCHAR(255),

  -- Metadata
  metadata JSONB,

  PRIMARY KEY (timestamp, run_id)  -- Partition column MUST be in PK for TimescaleDB
);

-- Create unique index for run_id lookups
CREATE UNIQUE INDEX idx_test_runs_run_id ON test_runs(run_id);

SELECT create_hypertable('test_runs', 'timestamp');

-- Indexes
CREATE INDEX idx_test_runs_commit ON test_runs(commit_sha, source, timestamp DESC);
CREATE INDEX idx_test_runs_repo_branch ON test_runs(repository, branch, timestamp DESC);
CREATE INDEX idx_test_runs_failures ON test_runs(timestamp DESC) WHERE failed_tests > 0;

-- Retention + compression
-- Retention + compression
-- Tiered retention strategy:
-- 1. Keep all runs for 7 days (high fidelity)
-- 2. Keep failures for 180 days (via retention policy on base table)
-- 3. Keep aggregate stats for 365 days (via continuous aggregates)

SELECT add_retention_policy('test_runs', INTERVAL '180 days'); -- Base retention (covers failures)
SELECT add_compression_policy('test_runs', INTERVAL '7 days'); -- Compress early

-- Continuous aggregate for long-term success trends (keep for 1 year)
CREATE MATERIALIZED VIEW test_runs_daily
WITH (timescaledb.continuous) AS
SELECT
  time_bucket('1 day', timestamp) AS bucket,
  repository,
  branch,
  source,
  count(*) as total_runs,
  sum(passed_tests) as total_passed,
  sum(failed_tests) as total_failed,
  avg(duration_ms) as avg_duration
FROM test_runs
GROUP BY bucket, repository, branch, source;

SELECT add_retention_policy('test_runs_daily', INTERVAL '365 days');

-- Test failures (detailed failure tracking with embeddings)
CREATE TABLE test_failures (
  timestamp TIMESTAMPTZ NOT NULL,
  failure_id BIGSERIAL,
  run_id VARCHAR(255) NOT NULL,

  -- Test details
  test_name VARCHAR(500) NOT NULL,
  test_file VARCHAR(500),
  suite_name VARCHAR(255),

  -- Error details
  error_type VARCHAR(255),
  error_message TEXT NOT NULL,
  error_stack TEXT,

  -- Location
  file_path VARCHAR(500),
  line_number INT,

  -- Embedding for similarity search
  error_embedding vector(1024),  -- BGE-large-en-v1.5

  PRIMARY KEY (timestamp, failure_id),

  CONSTRAINT fk_test_failures_run
    FOREIGN KEY (run_id)
    REFERENCES test_runs(run_id)
    ON DELETE CASCADE  -- Clean up failures when run deleted
);

SELECT create_hypertable('test_failures', 'timestamp');

-- Indexes
CREATE INDEX idx_test_failures_run ON test_failures(run_id, timestamp DESC);
CREATE INDEX idx_test_failures_test_name ON test_failures(test_name, timestamp DESC);

-- Vector similarity index (create AFTER hypertable)
CREATE INDEX idx_test_failures_embedding ON test_failures
USING ivfflat (error_embedding vector_cosine_ops)
WITH (lists = 100);

-- Retention + compression
SELECT add_retention_policy('test_failures', INTERVAL '180 days');
SELECT add_compression_policy('test_failures', INTERVAL '7 days');

-- Lint failures
CREATE TABLE lint_failures (
  timestamp TIMESTAMPTZ NOT NULL,
  failure_id BIGSERIAL,
  run_id VARCHAR(255) NOT NULL,

  -- Lint details
  rule VARCHAR(255) NOT NULL,
  severity VARCHAR(20) NOT NULL,
  message TEXT NOT NULL,

  -- Location
  file_path VARCHAR(500) NOT NULL,
  line INT,
  column INT,

  -- Source code (allowlist: only /src/, /tests/, /scripts/)
  source TEXT,

  PRIMARY KEY (timestamp, failure_id),

  CONSTRAINT fk_lint_failures_run
    FOREIGN KEY (run_id)
    REFERENCES test_runs(run_id)
    ON DELETE CASCADE
);

SELECT create_hypertable('lint_failures', 'timestamp');

-- Indexes
CREATE INDEX idx_lint_failures_run ON lint_failures(run_id, timestamp DESC);
CREATE INDEX idx_lint_failures_rule ON lint_failures(rule, timestamp DESC);

-- Retention + compression
SELECT add_retention_policy('lint_failures', INTERVAL '90 days');
SELECT add_compression_policy('lint_failures', INTERVAL '7 days');

-- Self-healing proposals (track fix attempts and outcomes)
CREATE TABLE selfheal_proposals (
  timestamp TIMESTAMPTZ NOT NULL,
  proposal_id VARCHAR(255) NOT NULL,

  -- Trigger
  trigger_type VARCHAR(50) NOT NULL,  -- 'test_failure', 'ci_failure', 'runtime_error', 'policy_violation'
  trigger_id VARCHAR(255),  -- Format: "run:{run_id}", "error:{error_id}", etc.

  -- Pattern tracking for SLO
  pattern_id VARCHAR(64) NOT NULL,  -- SHA256(canonicalized error + stack + files)

  -- Analysis
  similar_incidents JSONB,
  confidence_score DECIMAL(5,2),  -- 0-100

  -- Proposal
  description TEXT NOT NULL,
  diffs JSONB NOT NULL,
  tests_to_run JSONB,
  risk_score DECIMAL(5,2),  -- 0-100

  -- Outcome
  status VARCHAR(20) NOT NULL,  -- 'proposed', 'pr_opened', 'merged', 'rejected', 'failed_validation'
  pr_url VARCHAR(500),
  validation_results JSONB,

  -- Agent context (provider-agnostic)
  agent_model VARCHAR(100),  -- e.g., "claude-sonnet-4", "gpt-4", etc.
  agent_context JSONB,

  -- Embedding for learning
  proposal_embedding vector(1024),

  PRIMARY KEY (timestamp, proposal_id)
);

-- Create unique index for proposal_id lookups
CREATE UNIQUE INDEX idx_selfheal_proposal_id ON selfheal_proposals(proposal_id);

SELECT create_hypertable('selfheal_proposals', 'timestamp');

-- Pattern tracking index for SLO queries
CREATE INDEX idx_selfheal_pattern ON selfheal_proposals(pattern_id, timestamp DESC);
CREATE INDEX idx_selfheal_trigger ON selfheal_proposals(trigger_type, trigger_id);
CREATE INDEX idx_selfheal_status ON selfheal_proposals(status, timestamp DESC);

-- Vector similarity index (create AFTER hypertable)
CREATE INDEX idx_selfheal_proposal_embedding ON selfheal_proposals
USING ivfflat (proposal_embedding vector_cosine_ops)
WITH (lists = 100);

-- Retention + compression (keep longer for learning)
SELECT add_retention_policy('selfheal_proposals', INTERVAL '730 days');  -- 2 years
SELECT add_compression_policy('selfheal_proposals', INTERVAL '90 days');
```

### 2. Application Logs (@h-j-petroleum/toolkit-logger integration)

**Status:** ✅ Implemented - Logs with embeddings written to TimescaleDB via TimescaleTransport

```sql
CREATE TABLE application_logs (
  timestamp TIMESTAMPTZ NOT NULL,
  log_id BIGSERIAL,
  
  -- Source identification
  service VARCHAR(100) NOT NULL,
  environment VARCHAR(50) NOT NULL,
  hostname VARCHAR(255),
  
  -- Request context
  request_id VARCHAR(100),
  user_id VARCHAR(100),
  
  -- Log details
  level VARCHAR(20) NOT NULL,  -- trace, debug, info, warn, error, fatal
  message TEXT NOT NULL,
  logger_name VARCHAR(255),
  
  -- Structured context
  context JSONB,
  
  -- Error details (if level = error/fatal)
  error_type VARCHAR(255),
  error_message TEXT,
  error_stack TEXT,
  
  -- Embedding for semantic search
  message_embedding vector(1024),  -- BGE-large-en-v1.5 dimensions (generated via local embeddings-gpu service)
  
  PRIMARY KEY (timestamp, log_id)
);

SELECT create_hypertable('application_logs', 'timestamp');

-- Indexes
CREATE INDEX idx_logs_service_level ON application_logs(service, level, timestamp DESC);
CREATE INDEX idx_logs_request_id ON application_logs(request_id) WHERE request_id IS NOT NULL;
CREATE INDEX idx_logs_error ON application_logs(timestamp DESC) WHERE level IN ('error', 'fatal');

-- Vector similarity index for semantic search
CREATE INDEX idx_logs_embedding ON application_logs 
USING ivfflat (message_embedding vector_cosine_ops)
WITH (lists = 100);

-- Retention: 30 days for debug/trace, 90 days for others
SELECT add_retention_policy('application_logs', INTERVAL '90 days');
```

### 3. Runtime Metrics (@h-j-petroleum/toolkit-metrics integration)

**Status:** ⏳ Prometheus endpoints implemented; TimescaleMetricsExporter planned (not yet implemented)

```sql
CREATE TABLE metrics_samples (
  timestamp TIMESTAMPTZ NOT NULL,
  sample_id BIGSERIAL,
  
  -- Metric identification
  metric_name VARCHAR(255) NOT NULL,
  metric_type VARCHAR(20) NOT NULL,  -- counter, gauge, histogram
  
  -- Source
  service VARCHAR(100) NOT NULL,
  environment VARCHAR(50) NOT NULL,
  
  -- Labels/tags
  labels JSONB NOT NULL,
  
  -- Values
  value DOUBLE PRECISION,
  count BIGINT,  -- For histograms
  sum DOUBLE PRECISION,  -- For histograms
  
  PRIMARY KEY (timestamp, sample_id)
);

SELECT create_hypertable('metrics_samples', 'timestamp', 
  chunk_time_interval => INTERVAL '1 hour'
);

-- Indexes
CREATE INDEX idx_metrics_name_service ON metrics_samples(metric_name, service, timestamp DESC);
CREATE INDEX idx_metrics_labels ON metrics_samples USING gin(labels);

-- Continuous aggregate: 1-minute summaries
CREATE MATERIALIZED VIEW metrics_1min
WITH (timescaledb.continuous) AS
SELECT 
  time_bucket('1 minute', timestamp) AS bucket,
  metric_name,
  service,
  labels,
  AVG(value) as avg_value,
  MIN(value) as min_value,
  MAX(value) as max_value,
  COUNT(*) as sample_count
FROM metrics_samples
GROUP BY bucket, metric_name, service, labels;

-- Compress after 1 day
SELECT add_compression_policy('metrics_samples', INTERVAL '1 day');
SELECT add_retention_policy('metrics_samples', INTERVAL '30 days');
```

### 4. Distributed Traces (@h-j-petroleum/toolkit-telemetry integration)

**Status:** ✅ Implemented - OpenTelemetry traces captured and sent to observability stack

```sql
CREATE TABLE trace_spans (
  timestamp TIMESTAMPTZ NOT NULL,
  span_id VARCHAR(32) PRIMARY KEY,
  trace_id VARCHAR(32) NOT NULL,
  parent_span_id VARCHAR(32),
  
  -- Span details
  service VARCHAR(100) NOT NULL,
  operation_name VARCHAR(255) NOT NULL,
  duration_ms INT NOT NULL,
  
  -- Status
  status VARCHAR(20),  -- ok, error, unset
  status_message TEXT,
  
  -- Attributes
  attributes JSONB,
  
  -- Request context
  request_id VARCHAR(100),
  
  -- Events (logs within span)
  events JSONB
);

SELECT create_hypertable('trace_spans', 'timestamp');

CREATE INDEX idx_traces_trace_id ON trace_spans(trace_id, timestamp);
CREATE INDEX idx_traces_service ON trace_spans(service, timestamp DESC);
CREATE INDEX idx_traces_request_id ON trace_spans(request_id) WHERE request_id IS NOT NULL;
CREATE INDEX idx_traces_slow ON trace_spans(timestamp DESC) WHERE duration_ms > 1000;

SELECT add_retention_policy('trace_spans', INTERVAL '30 days');
```

### 5. Build & Deployment Events

```sql
CREATE TABLE build_events (
  timestamp TIMESTAMPTZ NOT NULL,
  event_id VARCHAR(255) PRIMARY KEY,
  
  -- Build identification
  repository VARCHAR(255) NOT NULL,
  branch VARCHAR(255) NOT NULL,
  commit_sha VARCHAR(40) NOT NULL,
  
  -- Event type
  event_type VARCHAR(50) NOT NULL,  -- build_start, build_complete, deploy_start, deploy_complete
  
  -- Outcome
  status VARCHAR(20) NOT NULL,  -- success, failure, cancelled
  
  -- Duration
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  duration_ms INT,
  
  -- Environment
  ci_provider VARCHAR(50),  -- github_actions, local
  environment VARCHAR(50),  -- production, staging, development
  
  -- Details
  artifacts JSONB,  -- URLs to build artifacts, docker images, etc.
  metadata JSONB,
  
  -- Embedding for semantic search of build errors
  error_embedding vector(1024)  -- BGE-large-en-v1.5 dimensions
);

SELECT create_hypertable('build_events', 'timestamp');

CREATE INDEX idx_builds_repo_branch ON build_events(repository, branch, timestamp DESC);
CREATE INDEX idx_builds_commit ON build_events(commit_sha);
CREATE INDEX idx_builds_status ON build_events(status, timestamp DESC);

SELECT add_retention_policy('build_events', INTERVAL '180 days');
```

### 6. Semantic Search Index (via vector-toolkit)

**Implementation:**
Reuse `vector-toolkit`'s `PostgresStore` schema but adapted for observability events.

```typescript
// Integration concept
import { PostgresStore } from '@vector-toolkit/stores-postgres';
import { OpenAIEmbeddingModel } from '@vector-toolkit/embeddings-openai';

// Point to hjps-ops local-llm service (FastAPI + ONNX Runtime on RTX 5090)
const embeddingModel = new OpenAIEmbeddingModel({
  apiKey: 'not-needed', // Local service doesn't require auth
  baseURL: 'http://local-llm:8000/v1', // hjps-ops local-llm container
  model: 'BAAI/bge-large-en-v1.5' // BGE model with 1024 dimensions
});

const store = new PostgresStore({
  connectionString: process.env.DATABASE_URL,
  tableName: 'observability_embeddings',
  dimensions: 1024 // BGE-large-en-v1.5 output dimensions
});
```

**Alternative: Direct Integration**
```typescript
// Direct HTTP call to local-llm service
async function generateEmbedding(text: string): Promise<number[]> {
  const response = await fetch('http://local-llm:8000/v1/embed', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ 
      texts: [text], 
      normalize: true 
    })
  });
  const data = await response.json();
  return data.embeddings[0]; // Returns 1024-dimensional vector
}
```

---

## 7. Coding Agent Service (vLLM)

**Service Definition:**
- **Name:** `vllm-chat`
- **Container:** `hjps-ops-vllm-chat`
- **Image:** `vllm/vllm-openai:latest`
- **Model:** `Qwen/Qwen2.5-32B-Instruct-AWQ`
- **Port:** `8000` (internal), `19000` (host)
- **API:** OpenAI-compatible Chat Completions (`/v1/chat/completions`)

**Capabilities:**
- **Tool Calling:** Native support for tool use (function calling)
- **Code Generation:** Specialized for coding tasks (32B parameters)
- **Context Window:** 32k tokens

**Integration:**
- **Base URL:** `http://vllm-chat:8000/v1`
- **Auth:** No API key required (internal)

---

## Integration Points

### 1. toolkit-logger Integration

**Status:** ✅ **Implemented** - `@h-j-petroleum/toolkit-logger@0.0.6` with TimescaleTransport

**Current Implementation:**
- toolkit-logger writes to stdout/files via Pino (default)
- TimescaleTransport writes logs to `application_logs` table in TimescaleDB
- Embeddings are generated via local OpenAI-compatible embedding service (default: `http://embeddings-1:8000`)
- Uses `POST /embed` endpoint with `{ texts: [...], normalize: true }` payload
- Implemented in `hjps-toolkit/packages/toolkit-logger/src/transports/timescale.ts`

**Configuration:**
```typescript
// Auto-configured when TIMESCALE_URL is present
import { createLogger } from '@h-j-petroleum/toolkit-logger';

const logger = createLogger({
  service: 'approval-api',
  level: 'info',
  // TimescaleTransport automatically enabled if TIMESCALE_URL env var is set
  // Embeddings generated via EMBEDDING_SERVICE_URL (default: http://embeddings-1:8000)
});
```

**See also:**
- Implementation: `hjps-toolkit/packages/toolkit-logger/src/transports/timescale.ts`
- Schema: `hjps-ops/observability/init-db/` (SQL init files)

### 2. toolkit-metrics Integration

**Status:** ⏳ **Planned** - Prometheus endpoints implemented, Timescale exporter planned

**Current Implementation:**
- `@h-j-petroleum/toolkit-metrics` exposes `/metrics` endpoints in services (implemented)
- Metrics are scraped by Prometheus (implemented)
- Services use `createMetrics()` or `PrometheusRegistry` API (see [toolkit-metrics README](../../prompts/repository-analysis/README.md))

**Planned Enhancement:** `TimescaleMetricsExporter` to write directly to `metrics_samples` table

**Note:** The `TimescaleMetricsExporter` is a planned exporter, not the current default. Current metrics flow is via Prometheus scraping. Timescale metrics ingestion will be layered on top later.

**Planned Implementation:**
```typescript
// packages/toolkit-metrics/src/exporters/TimescaleExporter.ts (PLANNED)
import { Pool } from 'pg';

export class TimescaleMetricsExporter {
  private pool: Pool;
  private flushInterval: NodeJS.Timeout;
  private buffer: MetricSample[] = [];
  
  constructor(config: TimescaleExporterConfig) {
    this.pool = new Pool(config);
    
    // Flush every 10 seconds
    this.flushInterval = setInterval(() => this.flush(), 10000);
  }
  
  record(metric: Metric, labels: Record<string, string>, value: number) {
    this.buffer.push({
      timestamp: new Date(),
      metricName: metric.name,
      metricType: metric.type,
      service: this.config.service,
      environment: this.config.environment,
      labels,
      value
    });
    
    if (this.buffer.length >= 1000) {
      this.flush();
    }
  }
  
  private async flush() {
    if (this.buffer.length === 0) return;
    
    const samples = this.buffer.splice(0);
    
    // Batch insert into metrics_samples table
    const values = samples.map((s, i) => 
      `($${i*7+1}, $${i*7+2}, $${i*7+3}, $${i*7+4}, $${i*7+5}, $${i*7+6}, $${i*7+7})`
    ).join(',');
    
    const params = samples.flatMap(s => [
      s.timestamp,
      s.metricName,
      s.metricType,
      s.service,
      s.environment,
      JSON.stringify(s.labels),
      s.value
    ]);
    
    await this.pool.query(`
      INSERT INTO metrics_samples (timestamp, metric_name, metric_type, service, environment, labels, value)
      VALUES ${values}
    `, params);
  }
}
```

### 3. toolkit-telemetry Integration

**Status:** ✅ **Implemented** - `@h-j-petroleum/toolkit-telemetry` exports to OpenTelemetry collectors

**Current Implementation:**
- toolkit-telemetry captures OpenTelemetry traces
- Traces are sent to the observability stack (OTEL collector → TimescaleDB)
- Configured in hjps-ops services via environment variables

```typescript
// hjps-frontend/src/lib/otel.server.ts
// Already configured! Just set env var:
// TELEMETRY_EXPORTER_OTLP_URL=http://otel-collector:4318/v1/traces
```

---

## Phase 3.5: Agent Integration (AgentPort)

**Goal:** Standardize how agents interact with the system via `vllm-chat`.

### 1. AgentPort Interface
Defined in `toolkit-standards/src/ports/AgentPort.ts`:

```typescript
export interface AgentPort {
  chat(messages: ChatMessage[], tools?: ToolDefinition[]): Promise<ChatResponse>;
  generateCode(prompt: string, context: CodeContext): Promise<CodeBlock>;
}
```

### 2. VLLMAgentAdapter
Implementation in `toolkit-standards/src/adapters/VLLMAgentAdapter.ts`:

```typescript
export class VLLMAgentAdapter implements AgentPort {
  constructor(private baseUrl: string = 'http://vllm-chat:8000/v1') {}
  // ... implements OpenAI client calls to vLLM ...
}
```

---

## Agent Access Layer (MCP Tools)

### MCP Server for Observability

```typescript
// packages/observability-mcp-server/src/index.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { Pool } from 'pg';
import { OpenAI } from 'openai';

const server = new Server({
  name: 'observability-server',
  version: '1.0.0',
});

const db = new Pool({
  host: process.env.TIMESCALE_HOST,
  database: 'observability',
  // ...
});

const openai = new OpenAI();

// Tool 1: Search error logs semantically
server.tool(
  'search_errors',
  'Search for similar errors using natural language',
  {
    query: {
      type: 'string',
      description: 'Natural language description of the error',
    },
    timeRange: {
      type: 'string',
      description: 'Time range (e.g., "24h", "7d")',
      default: '24h'
    },
    service: {
      type: 'string',
      description: 'Filter by service name (optional)'
    }
  },
  async ({ query, timeRange, service }) => {
    // Generate embedding for query
    const embedding = await openai.embeddings.create({
      model: 'text-embedding-ada-002',
      input: query
    });
    
    // Search similar errors
    const result = await db.query(`
      SELECT * FROM search_similar_events(
        $1::vector,
        0.7,
        10
      )
      WHERE timestamp > NOW() - $2::interval
      ${service ? 'AND service = $3' : ''}
      ORDER BY similarity DESC
    `, [
      JSON.stringify(embedding.data[0].embedding),
      timeRange,
      service
    ]);
    
    return {
      content: [{
        type: 'text',
        text: JSON.stringify(result.rows, null, 2)
      }]
    };
  }
);

// Tool 2: Compare test runs
server.tool(
  'compare_test_runs',
  'Compare local and CI test results for a commit',
  {
    commitSha: {
      type: 'string',
      description: 'Git commit SHA'
    }
  },
  async ({ commitSha }) => {
    const result = await db.query(`
      SELECT 
        l.run_id as local_run_id,
        c.run_id as ci_run_id,
        l.passed_tests as local_passed,
        c.passed_tests as ci_passed,
        l.failed_tests as local_failed,
        c.failed_tests as ci_failed,
        l.node_version as local_node,
        c.node_version as ci_node,
        l.lockfile_hash as local_lockfile,
        c.lockfile_hash as ci_lockfile
      FROM test_runs l
      JOIN test_runs c ON l.commit_sha = c.commit_sha
      WHERE l.source = 'local' 
        AND c.source = 'ci'
        AND l.commit_sha = $1
    `, [commitSha]);
    
    if (result.rows.length === 0) {
      return {
        content: [{
          type: 'text',
          text: 'No matching test runs found for this commit'
        }]
      };
    }
    
    const comparison = result.rows[0];
    const differences = [];
    
    if (comparison.local_passed !== comparison.ci_passed) {
      differences.push(`Test count mismatch: ${comparison.local_passed} passed locally vs ${comparison.ci_passed} in CI`);
    }
    
    if (comparison.local_node !== comparison.ci_node) {
      differences.push(`Node version: ${comparison.local_node} (local) vs ${comparison.ci_node} (CI)`);
    }
    
    if (comparison.local_lockfile !== comparison.ci_lockfile) {
      differences.push('Different package-lock.json hashes - dependency drift detected');
    }
    
    return {
      content: [{
        type: 'text',
        text: `## Test Comparison for ${commitSha}\n\n` +
              `### Summary\n` +
              `- Local: ${comparison.local_passed}/${comparison.local_passed + comparison.local_failed} passed\n` +
              `- CI: ${comparison.ci_passed}/${comparison.ci_passed + comparison.ci_failed} passed\n\n` +
              (differences.length > 0 
                ? `### Differences Found\n${differences.map(d => `- ${d}`).join('\n')}\n`
                : `### ✅ No significant differences detected\n`)
      }]
    };
  }
);

// Tool 3: Trace request across services
server.tool(
  'trace_request',
  'Follow a request across all services and logs',
  {
    requestId: {
      type: 'string',
      description: 'Request ID to trace'
    }
  },
  async ({ requestId }) => {
    // Get all spans for this request
    const spans = await db.query(`
      SELECT * FROM trace_spans
      WHERE request_id = $1
      ORDER BY timestamp ASC
    `, [requestId]);
    
    // Get all logs for this request
    const logs = await db.query(`
      SELECT * FROM application_logs
      WHERE request_id = $1
      ORDER BY timestamp ASC
    `, [requestId]);
    
    return {
      content: [{
        type: 'text',
        text: `## Request Trace: ${requestId}\n\n` +
              `### Spans (${spans.rows.length})\n` +
              spans.rows.map(s => 
                `- ${s.service}.${s.operation_name} (${s.duration_ms}ms) ${s.status !== 'ok' ? '❌ ' + s.status_message : ''}`
              ).join('\n') +
              `\n\n### Logs (${logs.rows.length})\n` +
              logs.rows.map(l =>
                `[${l.timestamp}] ${l.level.toUpperCase()} ${l.service}: ${l.message}`
              ).join('\n')
      }]
    };
  }
);

// Tool 4: Get service health metrics
server.tool(
  'service_health',
  'Get current health metrics for a service',
  {
    service: {
      type: 'string',
      description: 'Service name'
    },
    timeRange: {
      type: 'string',
      description: 'Time range (default: 1h)',
      default: '1h'
    }
  },
  async ({ service, timeRange }) => {
    const metrics = await db.query(`
      SELECT 
        metric_name,
        AVG(value) as avg_value,
        MAX(value) as max_value,
        MIN(value) as min_value,
        COUNT(*) as sample_count
      FROM metrics_samples
      WHERE service = $1
        AND timestamp > NOW() - $2::interval
      GROUP BY metric_name
      ORDER BY metric_name
    `, [service, timeRange]);
    
    const errorCount = await db.query(`
      SELECT COUNT(*) as error_count
      FROM application_logs
      WHERE service = $1
        AND level IN ('error', 'fatal')
        AND timestamp > NOW() - $2::interval
    `, [service, timeRange]);
    
    return {
      content: [{
        type: 'text',
        text: `## Service Health: ${service}\n\n` +
              `### Metrics (${timeRange})\n` +
              metrics.rows.map(m =>
                `- ${m.metric_name}: avg=${m.avg_value.toFixed(2)}, max=${m.max_value.toFixed(2)}, min=${m.min_value.toFixed(2)}`
              ).join('\n') +
              `\n\n### Errors\n` +
              `${errorCount.rows[0].error_count} errors in the last ${timeRange}`
      }]
    };
  }
);
```

---

## Grafana Dashboards (Expanded)

### Dashboard 1: Development Overview

**Panels:**
1. Test Pass Rate (Local vs CI) - Time series
2. Active Deployments - Timeline
3. Service Health Heatmap - Services × Error rate
4. Recent Errors - Table with semantic grouping
5. Build Duration Trend - Time series
6. Code Coverage Trend - Time series

### Dashboard 2: Service Deep Dive

**Panels:**
1. Request Rate (from metrics)
2. P95 Latency (from metrics)
3. Error Rate (from logs + metrics)
4. Active Traces - Table
5. Slow Queries - Table (from trace spans)
6. Recent Deployments - Timeline

### Dashboard 3: Test Intelligence

**Panels:**
1. Flaky Tests - Tests that intermittently fail
2. Slowest Tests - Duration percentiles
3. Test Coverage Heat map - By module
4. Local/CI Mismatch Rate - Percentage over time
5. Most Common Failures - Top 10 errors

---

## Data Lifecycle Management

### Retention Policies

```sql
-- Hot data: Last 7 days (uncompressed, fast queries)
-- Warm data: 8-30 days (compressed, slightly slower queries)
-- Cold data: 31-90 days (highly compressed, archival queries)
-- Deleted: > 90 days (except aggregates which persist)

-- Application logs
SELECT add_retention_policy('application_logs', INTERVAL '90 days');
SELECT add_compression_policy('application_logs', INTERVAL '7 days');

-- Metrics (keep longer for trend analysis)
SELECT add_retention_policy('metrics_samples', INTERVAL '180 days');
SELECT add_compression_policy('metrics_samples', INTERVAL '1 day');

-- Traces (expensive, shorter retention)
SELECT add_retention_policy('trace_spans', INTERVAL '30 days');
SELECT add_compression_policy('trace_spans', INTERVAL '7 days');

-- Test results (keep longer for historical analysis)
SELECT add_retention_policy('test_runs', INTERVAL '365 days');
SELECT add_compression_policy('test_runs', INTERVAL '30 days');

-- Build events (keep for compliance)
SELECT add_retention_policy('build_events', INTERVAL '730 days');  -- 2 years
SELECT add_compression_policy('build_events', INTERVAL '90 days');
```

### Continuous Aggregates (Pre-computed Views)

```sql
-- Daily rollups persist forever (small data size)
-- Used for long-term trend analysis

CREATE MATERIALIZED VIEW daily_service_health
WITH (timescaledb.continuous) AS
SELECT 
  time_bucket('1 day', timestamp) AS day,
  service,
  environment,
  COUNT(*) FILTER (WHERE level IN ('error', 'fatal')) as error_count,
  COUNT(*) as total_logs,
  COUNT(DISTINCT request_id) as unique_requests
FROM application_logs
GROUP BY day, service, environment;

CREATE MATERIALIZED VIEW daily_test_summary
WITH (timescaledb.continuous) AS
SELECT 
  time_bucket('1 day', timestamp) AS day,
  repository,
  branch,
  source,
  COUNT(*) as test_runs,
  AVG(passed_tests::float / NULLIF(total_tests, 0)) * 100 as avg_pass_rate,
  SUM(CASE WHEN failed_tests > 0 THEN 1 ELSE 0 END) as runs_with_failures
FROM test_runs
GROUP BY day, repository, branch, source;
```

---

## Implementation Phases

### Phase 1: Infrastructure ✅ **COMPLETE** (Week 1)
- [x] ~~Update `hjps-ops/docker-compose.dev-stack.yml`~~ Created `docker-compose.observability.yml` to include:
  - [x] TimescaleDB (with pgvector)
  - [x] Grafana with provisioned datasources
  - [x] OTEL Collector (configured for traces, metrics, logs)
  - [x] Ollama (for alternative embedding models)
  - [x] **Custom GPU Embedding Service** (`embeddings-gpu`) - FastAPI + ONNX Runtime (deployed as `embeddings-1` container)
- [x] Configure networking between services and observability stack
- [x] Verify GPU access for embedding container (CUDA 12.6 + cuDNN 9)

**Actual Implementation:**
- Built production-grade FastAPI service at `services/embeddings-gpu/`
- Model: `BAAI/bge-large-en-v1.5` (1024 dimensions)
- ONNX Runtime with CUDAExecutionProvider
- Endpoint: `POST /embed` with batch support (up to 32 texts)
- Health check: `GET /healthz`
- GPU detection with CPU fallback
- Deployed as `embeddings-1` container in hjps-ops stack

### Phase 2: Vector Integration (Week 2) - **✅ COMPLETE**
- [x] Create `pgvector` schema for logs/metrics/traces
- [x] Update `@h-j-petroleum/toolkit-logger` to support vector generation
- [x] Configure `embeddings-gpu` service for batch processing
- [x] Integrate `@h-j-petroleum/toolkit-logger` with `embeddings-gpu` via `TimescaleTransport`
- [x] Published `@h-j-petroleum/toolkit-logger@0.0.6` to GitHub Packages
- [x] Removed Docker stubbing logic to use real published packages
- [x] Verified end-to-end: logs → embeddings → TimescaleDB storage

**Completed Work:**
1. ✅ Database schema updated to use `vector(1024)` for BGE embeddings
2. ✅ `TimescaleTransport` created with batching and error resilience
3. ✅ Embeddings generation integrated via custom GPU service
4. ✅ End-to-end verified: 41 logs with 41 embeddings successfully stored
5. ✅ Refactored `toolkit-logger` to TypeScript with comprehensive JSDoc
6. ✅ Fixed Docker build configurations and database connection settings

### Phase 3: Service Adoption (Week 3) - **✅ COMPLETE**
- [x] Configure `hjps-ops` services (approval-api, pdf-generator, timesheet-api, api-gateway) to emit logs with embeddings
- [x] Upgrade to `@h-j-petroleum/toolkit-logger@0.0.6` with structured logging fixes
- [x] Verify end-to-end log visibility with vector embeddings
- [x] Embeddings generated via local OpenAI-compatible service (`http://embeddings-1:8000`), not direct OpenAI API
- [x] Create initial Grafana dashboards for log visualization (6 dashboards provisioned)
- [ ] Configure `hjps-frontend` telemetry env vars (deferred to Phase 4)

> **Note:** hjps-frontend browser telemetry moved to a separate phase after GUI development.

---

## Implementation Phases (Phase 7 Transition)

**Phase 1-3: Foundation (Within Phase 6 of Overall Standards Initiative)**
- Phase 1: Infrastructure ✅ Complete (Week 1)
- Phase 2: Vector Integration ✅ Complete (Week 2)
- Phase 3: Services ✅ Complete (Week 3)
  - All hjps-ops services (api-gateway, approval-api, pdf-generator, timesheet-api)
    logging to TimescaleDB with embeddings via @h-j-petroleum/toolkit-logger@0.0.6

**Phase 4-6: Self-Healing (Explicitly Phase 7 of Overall Standards Initiative)**
- ⏳ Phase 4: CI Observability + Semantic Search
  - ⏳ CI schema implemented (local tested, CI verification pending)
  - ⏳ Grafana dashboards provisioned (pending verification with real CI data)
  - ⏳ Reporter script implemented (local tested, CI integration pending)
  - ⏳ Pending: MCP tools (search_errors, compare_test_runs, etc.)
  - ⏳ Pending: Vector similarity search functions
- ⏳ Phase 5: Agent Access (MCP server, SelfHealingService)
- ⏳ Phase 6: Apply & Monitor (worker, Grafana dashboards)

**Rationale for Separation:**
Phase 6 of the overall standards initiative = 4 weeks (Weeks 1-4).
Self-healing = 3 additional weeks (Weeks 5-7).
Explicit separation as "Phase 7" prevents Phase 6 scope creep and allows Phase 6 to complete with clear acceptance criteria.

---

### Phase 4: CI Observability + Semantic Search (Week 4) - **⏳ IN PROGRESS**
- [x] Complete test_runs, test_failures, lint_failures schema implementation (`hjps-ops/observability/init-db/04-ci-schema.sql`)
- [x] Add vector embeddings to test failures (1024-dim BGE-large-en-v1.5)
- [x] Create CI reporter script (`hjps-ops/scripts/ci-log-reporter.mjs`)
- [x] Create Grafana dashboards (ci-test-intelligence, build-deploy-events, self-healing-overview)
- [x] Update GitHub Actions workflow to capture test results
- [ ] Verify CI integration end-to-end (pending CI run)
- [ ] Implement similarity search for test failures
- [ ] Build compare_test_runs MCP tool

#### hjps-ops CI Observability Implementation (2025-12-01)

**Schema:** `observability/init-db/04-ci-schema.sql`
- `test_runs` - Local vs CI comparison with environment tracking
- `test_failures` - Individual failures with 1024-dim embeddings
- `lint_failures` - TypeScript/ESLint violations
- `selfheal_proposals` - Self-healing fix proposals with pattern tracking
- Continuous aggregates: `test_runs_daily`, `selfheal_daily`

**Reporter:** `scripts/ci-log-reporter.mjs`
- Parses vitest JSON output + tsc errors
- Generates embeddings via embeddings-gpu service
- Reports to TimescaleDB

**GitHub Actions:** `.github/workflows/reusable-test-setup.yml`
- Captures vitest JSON + tsc output as artifacts
- Results available for local sync or future webhook integration

**Dashboards:** 3 new Grafana dashboards provisioned
- `ci-test-intelligence` - Pass rates, flaky tests, local vs CI comparison
- `build-deploy-events` - Build duration, success rate, deployment tracking
- `self-healing-overview` - Proposal tracking, acceptance rate, pattern analysis

**Deliverables:**
- Test observability fully operational
- Historical test data queryable
- Flaky test detection working

### Phase 5: Agent Access & Self-Healing Proposal (Week 5) - **⏳ PHASE 7**
- [ ] Build MCP server with observability tools
- [ ] Implement search_errors tool
- [ ] Implement compare_test_runs tool
- [ ] Implement trace_request tool
- [ ] Implement service_health tool
- [ ] Implement `selfheal:detect` skill
- [ ] Implement `selfheal:retrieve` skill (similarity search)
- [ ] Implement `selfheal:propose` skill (agent-generated fixes)
- [ ] Add SelfHealingService to standards-cli
- [ ] Define ports: `Telemetry`, `EmbeddingStore`, `SelfHealer`
- [ ] Add logic layer: pure scoring/grouping/decision functions
- [ ] Test with agents (Claude, Gemini)

**Deliverables:**
- MCP tools accessible to agents
- Self-healing proposals generated
- Ports/services architecture in place
- **See:** [DevOS Canvas §6 Self-Healing](../../DEVOS_ARCHITECTURE_CANVAS.md#6-self-healing--failure-handling)

### Phase 6: Apply, Validate & Monitor (Week 6) - **⏳ PHASE 7**
- [ ] Implement `selfheal:apply` skill
- [ ] Add worker/daemon entrypoint
- [ ] Wire up CI webhook subscriptions
- [ ] Deploy self-healing Grafana dashboards
- [ ] Run pilot with hjps-ops (advisory mode, autonomy Level 0)
- [ ] Validate guardrails (no silent mutations, SLO limits, branch naming)
- [ ] Deploy to all services
- [ ] Train team on dashboards
- [ ] Document MCP tools for agents
- [ ] Set up alerting
- [ ] Measure MTTR improvement
- [ ] Monitor and optimize

**Deliverables:**
- End-to-end self-healing loop operational
- PRs opened automatically for validated fixes
- Metrics dashboard showing effectiveness
- Zero policy violations by self-heal system

---

## Technology Stack

**Storage:**
- PostgreSQL 16+ with TimescaleDB and pgvector extensions
- Hosting: Neon (serverless), Supabase, or self-hosted Docker

**Collection:**
- Enhanced @h-j-petroleum/standards-cli
- toolkit-logger + TimescaleDB transport
- toolkit-metrics + TimescaleDB exporter
- toolkit-telemetry + TimescaleDB span exporter

**Query:**
- Direct SQL for structured queries
- Vector similarity for semantic search
- GraphQL API (optional, for web UI)

**Visualization:**
- Grafana for dashboards and alerting
- Custom Next.js web UI for deep investigation

**Agent Access:**
- MCP server with observability tools
- Direct SQL access via secure API
- Vector embeddings via local OpenAI-compatible service (embeddings-gpu, not direct OpenAI API)

---

## Cost Estimation

**Self-Hosted (RTX 5090):** $0/month
- **Database:** TimescaleDB + pgvector (Docker)
- **Embeddings:** Local GPU inference (Ollama/TEI)
- **Visualization:** Grafana (Docker)
- **Hardware:** Existing RTX 5090 (Primary), Ubuntu Server (Redundancy)

**Note on Redundancy:**
- Primary (RTX 5090): Fast embedding generation
- Secondary (Ubuntu Server): CPU-based inference (slower but functional) or remote call to Primary


---

## Success Metrics

- [ ] 95%+ observability completeness (all services instrumented)
- [ ] < 30 seconds to locate root cause using semantic search
- [ ] Agents can debug issues autonomously 80% of the time
```

---

## Grafana Dashboards (Expanded)

### Dashboard 1: Development Overview

**Panels:**
1. Test Pass Rate (Local vs CI) - Time series
2. Active Deployments - Timeline
3. Service Health Heatmap - Services × Error rate
4. Recent Errors - Table with semantic grouping
5. Build Duration Trend - Time series
6. Code Coverage Trend - Time series

### Dashboard 2: Service Deep Dive

**Panels:**
1. Request Rate (from metrics)
2. P95 Latency (from metrics)
3. Error Rate (from logs + metrics)
4. Active Traces - Table
5. Slow Queries - Table (from trace spans)
6. Recent Deployments - Timeline

### Dashboard 3: Test Intelligence

**Panels:**
1. Flaky Tests - Tests that intermittently fail
2. Slowest Tests - Duration percentiles
3. Test Coverage Heat map - By module
4. Local/CI Mismatch Rate - Percentage over time
5. Most Common Failures - Top 10 errors

---

## Data Lifecycle Management

### Retention Policies

```sql
-- Hot data: Last 7 days (uncompressed, fast queries)
-- Warm data: 8-30 days (compressed, slightly slower queries)
-- Cold data: 31-90 days (highly compressed, archival queries)
-- Deleted: > 90 days (except aggregates which persist)

-- Application logs
SELECT add_retention_policy('application_logs', INTERVAL '90 days');
SELECT add_compression_policy('application_logs', INTERVAL '7 days');

-- Metrics (keep longer for trend analysis)
SELECT add_retention_policy('metrics_samples', INTERVAL '180 days');
SELECT add_compression_policy('metrics_samples', INTERVAL '1 day');

-- Traces (expensive, shorter retention)
SELECT add_retention_policy('trace_spans', INTERVAL '30 days');
SELECT add_compression_policy('trace_spans', INTERVAL '7 days');

-- Test results (keep longer for historical analysis)
SELECT add_retention_policy('test_runs', INTERVAL '365 days');
SELECT add_compression_policy('test_runs', INTERVAL '30 days');

-- Build events (keep for compliance)
SELECT add_retention_policy('build_events', INTERVAL '730 days');  -- 2 years
SELECT add_compression_policy('build_events', INTERVAL '90 days');
```

### Continuous Aggregates (Pre-computed Views)

```sql
-- Daily rollups persist forever (small data size)
-- Used for long-term trend analysis

CREATE MATERIALIZED VIEW daily_service_health
WITH (timescaledb.continuous) AS
SELECT 
  time_bucket('1 day', timestamp) AS day,
  service,
  environment,
  COUNT(*) FILTER (WHERE level IN ('error', 'fatal')) as error_count,
  COUNT(*) as total_logs,
  COUNT(DISTINCT request_id) as unique_requests
FROM application_logs
GROUP BY day, service, environment;

CREATE MATERIALIZED VIEW daily_test_summary
WITH (timescaledb.continuous) AS
SELECT 
  time_bucket('1 day', timestamp) AS day,
  repository,
  branch,
  source,
  COUNT(*) as test_runs,
  AVG(passed_tests::float / NULLIF(total_tests, 0)) * 100 as avg_pass_rate,
  SUM(CASE WHEN failed_tests > 0 THEN 1 ELSE 0 END) as runs_with_failures
FROM test_runs
GROUP BY day, repository, branch, source;
```

---

## Implementation Phases

### Phase 1: Infrastructure (Week 1)
- [ ] Update `hjps-ops/docker-compose.dev-stack.yml` to include:
  - TimescaleDB (with pgvector)
  - Grafana

### Phase 4: Semantic Search (Week 4)
- [ ] Set up OpenAI API integration
- [ ] Implement embedding generation for errors
- [ ] Create vector similarity search functions
- [ ] Test semantic search queries

### Phase 5: Agent Access (Week 5)
- [ ] Build MCP server with tools
- [ ] Implement search_errors tool
- [ ] Implement compare_test_runs tool
- [ ] Implement trace_request tool
- [ ] Implement service_health tool
- [ ] Test with Gemini agent

### Phase 6: Rollout (Week 6)
- [ ] Deploy to all services
- [ ] Train team on dashboards
- [ ] Document MCP tools for agents
- [ ] Set up alerting
- [ ] Monitor and optimize

### Phase 7: Self-Healing & Shadow Mode (Week 7+)
- [ ] Implement "Shadow Mode" (Level 0.5)
  - [ ] Agents generate proposals but do NOT apply them
  - [ ] Proposals are logged to `selfheal_proposals` with `status='shadow'`
  - [ ] Offline grading tool for human review of shadow proposals
- [ ] Enable "Level 0" (Suggest Only)
  - [ ] Once confidence > 90% in shadow mode
  - [ ] Agents open PRs but require human merge

### Frontend Observability Integration (Parallel to Phase 6-7)

**Context:** Once `frontend-toolkit` and GUI apps are built (Week 3-4), browser telemetry completes the observability loop.

**Phase 6 Week 3-4:**
- [ ] Add browser telemetry exports to `frontend-toolkit`
  - [ ] OpenTelemetry Web SDK wrapper
  - [ ] Auto-instrumentation for fetch, navigation, errors
  - [ ] TraceparentManagement for distributed tracing
- [ ] Instrument `approval-portal` and `timesheet-portal`
  - [ ] Page load performance metrics
  - [ ] User interaction events (clicks, form submissions)
  - [ ] Client-side errors (unhandled exceptions, network failures)
- [ ] Feed client-side data → TimescaleDB
  - [ ] Reuse existing `application_logs` + `traces` tables
  - [ ] Add `client_errors` field to `application_logs`

**Phase 7 Integration:**
- Self-healing retrieves both backend + frontend failures
- Vector similarity search across server-side logs + client-side errors
- Agentic fixes consider full-stack context

---

## Technology Stack

**Storage:**
- PostgreSQL 16+ with TimescaleDB and pgvector extensions
- Hosting: Neon (serverless), Supabase, or self-hosted Docker

**Collection:**
- Enhanced @h-j-petroleum/standards-cli
- toolkit-logger + TimescaleDB transport
- toolkit-metrics + TimescaleDB exporter
- toolkit-telemetry + TimescaleDB span exporter

**Query:**
- Direct SQL for structured queries
- Vector similarity for semantic search
- GraphQL API (optional, for web UI)

**Visualization:**
- Grafana for dashboards and alerting
- Custom Next.js web UI for deep investigation

**Agent Access:**
- MCP server with observability tools
- Direct SQL access via secure API
- Vector embeddings via local OpenAI-compatible service (embeddings-gpu, not direct OpenAI API)

---

## Cost Estimation

**Self-Hosted (RTX 5090):** $0/month
- **Database:** TimescaleDB + pgvector (Docker)
- **Embeddings:** Local GPU inference (Ollama/TEI)
- **Visualization:** Grafana (Docker)
- **Hardware:** Existing RTX 5090 (Primary), Ubuntu Server (Redundancy)

**Note on Redundancy:**
- Primary (RTX 5090): Fast embedding generation
- Secondary (Ubuntu Server): CPU-based inference (slower but functional) or remote call to Primary


---

## Success Metrics

- [ ] 95%+ observability completeness (all services instrumented)
- [ ] < 30 seconds to locate root cause using semantic search
- [ ] Agents can debug issues autonomously 80% of the time
- [ ] Zero "worked locally" merge friction
- [ ] < 5 minute time to first insight on production incidents
- [ ] 100% test runs logged (local + CI)
- [ ] 100% error logs with embeddings
- [ ] 100% requests traced end-to-end

---

## Integration with AI-Native DevOS

**Document Relationship:**
- **This Document:** Canonical source for observability database schemas (CREATE TABLE statements)
- **DevOS Canvas:** Canonical source for architecture patterns, ports, and self-healing state machine
- **Relationship:** This platform implements the `Telemetry` and `EmbeddingStore` ports defined in the canvas

**Ports Implemented:**
- `Telemetry` → TimescaleDB adapter ([DevOS Canvas §5.1](../../DEVOS_ARCHITECTURE_CANVAS.md#51-telemetry-model))
- `EmbeddingStore` → pgvector + GPU embeddings service ([DevOS Canvas §5.2](../../DEVOS_ARCHITECTURE_CANVAS.md#52-embeddings--vector-store))
- `SelfHealer` → ⏳ Planned for Phase 5-6 (Phase 7 overall) ([DevOS Canvas §6](../../DEVOS_ARCHITECTURE_CANVAS.md#6-self-healing--failure-handling))

**See Also:**
- [AI-Native DevOS Architecture Canvas](../../DEVOS_ARCHITECTURE_CANVAS.md)
- [AI-Native DevOS Vision](../../AI_NATIVE_DEVOS_VISION.md) (strategic context)

---

## Future Policy Alignment (Phase 7+)

This infrastructure directly enables the following future policy skills identified in `docs/planning/analysis/recommendations/skill-roadmap.md`:

1.  **`obs:otel-check`**: Will validate that services are sending traces to the OTEL Collector established here.
2.  **`obs:attribute-check`**: Will validate that traces contain required attributes (service.name, environment) stored in TimescaleDB.
3.  **`cost:embedding-budget`**: Will query `metrics_samples` (token usage) to enforce budget limits, even for local models.

---
```
