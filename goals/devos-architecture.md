# AI-Native DevOS Architecture Canvas

Last Updated: 2025-12-03
Policy-Version: 1.0.0
TTL: 90d
Owner: Architecture + Operations
Status: Foundation Complete (Phases 1-2) ✅ | Self-Healing Planned ⏳

---

## What This Is

**AI-Native Development Operating System (DevOS):**

An architecture + policy framework that acts as a development operating system for building and evolving serious systems when most of the code is touched by AI agents.

**Not just a web framework or DevOps tooling. A comprehensive development OS.**

This is the operating system for how all code in our universe is:
- **Structured** (hexagonal + functional core)
- **Tested** (pure logic, boundary-only mocking, deterministic)
- **Observed** (time-series telemetry + vector memory)
- **Reasoned About** (contracts, schemas, policy)
- **Modified** (by humans OR agents, with guardrails)

### The Problem We're Solving

**Core Question:**
> How do you build and evolve serious systems when most of the code is touched by AI agents?

**What AI Didn't Solve:**
- ❌ Inconsistent architecture (without governance)
- ❌ Long-term context loss (forgets past decisions)
- ❌ Fragile at scale (without strong rails)
- ❌ No domain understanding (without policies)
- ❌ "Vibe-driven patches" (without structure)

**What DevOS Provides:**
- ✅ Stable center (pure logic AI can't easily corrupt)
- ✅ Explicit boundaries (ports/adapters isolate I/O)
- ✅ Telemetry + time (system can see its own behavior)
- ✅ Vector memory (remembers patterns and past fixes)
- ✅ Policy enforcement (agents can't silently break rules)
- ✅ Self-healing workflows (failures → guided fixes)

### Commercial Use

**Near-Term: AI-Native Dev Studio**
- Use DevOS to deliver software projects faster, safer, smaller teams
- New projects: start from DevOS template, immediate structure/tests/telemetry
- Maintenance: self-healing + telemetry + vector memory cut bugfix time
- Sell: **outcomes** (internal tools, APIs, automation, AI integrations)
- DevOS is your **leverage engine**, not your first product

**Long-Term: DevOS Cloud (Phase 2+)**
- Multi-tenant service: telemetry ingestion, vector memory, self-healing orchestration
- Dashboards, PR bots, policy controls
- Open standards + hosted platform (separate concerns)
- **Only after internal studio model is successful**

### Current Implementation Status (Reference)

As of this writing (proof of concept / foundation):
- ✅ Packages published to GitHub Packages under `@h-j-petroleum` scope
- ✅ @h-j-petroleum/toolkit-logger@0.0.6 with TimescaleDB transport and vector embeddings
- ✅ Logs with 1024-dim embeddings verified (end-to-end test across hjps-ops services)
- ✅ 19 skills in MCP catalog (see `mcp/skills/*.yml` for canonical list)

**Note:** These are current reference implementations. Numbers and versions will evolve as the system matures.

### North Star

> A world where the majority of routine coding, bugfixing, and maintenance is performed by agents running on a well-structured, observable, policy-enforced architecture—and your role is to set direction, model domains, and make high-level tradeoffs.

---

## Document Structure & Canonical Ownership

**This canvas is canonical for:**
- Architecture patterns (hexagonal + functional core)
- Purity rules (logic layer constraints)
- DI patterns (service factories)
- Port definitions (Telemetry, EmbeddingStore, SelfHealer, AgentPort)
- Autonomy levels (0-3 ladder)
- Repo tiers (0-2 requirements)
- Self-healing state machine (detect → retrieve → propose → apply)

**Other documents are canonical for:**
- **Telemetry/test schemas:** `observability/IMPLEMENTATION_PLAN.md` (this document links there)
- **Phase 6 tasks:** `PHASE6_TASKS.md` (detailed checklist)
- **Policy skills:** `POLICY_REFERENCE.md` (skill catalog)
- **Dependencies:** `policy/node/constraints.json` (floors, bans)
- **Dependency rules:** `packages/standards-cli/assets/dep-cruiser.config.cjs`

**Cross-Reference Policy:**
- Each major doc: 1-2 "upward" references to canvas, 1-2 "sideways" to sibling docs
- Use **section anchors** (`#workstream-4`), NOT line numbers
- Avoid dozens of references (maintainability)

---

## 1. Purpose & Scope

### 1.1 Purpose

Define the core architecture, policies, and implementation for **DevOS**—an architecture + policy framework that acts as a development operating system, standardizing how repos are:
- **Structured:** Hexagonal architecture (contracts/logic/ports/services/adapters/entrypoints)
- **Tested:** Pure logic, deterministic behavior, boundary-only mocking
- **Observed:** Time-series telemetry + vector memory for incident retrieval
- **Healed:** Agent-proposed fixes with policy validation

### 1.2 Scope

**Applies to:** Repos adopting DevOS (via `@h-j-petroleum/standards-cli`)

**Covers:**
- Architecture layers & dependency rules
- Enforcement tooling (local gates + CI gates)
- Telemetry + vector memory (TimescaleDB + pgvector)
- Self-healing loop (detect → retrieve → propose → apply)
- Autonomy levels (suggest-only → auto-fix → auto-remediation)
- Repo tiers (lightweight → standard → critical)

**Excludes:** Business/product-specific logic (only systemic architecture)

### 1.3 Why Build This Now

**AI is accelerating, but organizational architecture hasn't caught up:**
- Most teams: 2010-era patterns (ad hoc MVC)
- Adding: 2025-era AI agents
- **Mismatch will break things**

**By building AI-Native DevOS now:**
- ✅ Defensible internal advantage (deliver more with less)
- ✅ Future-proof systems against chaotic AI usage
- ✅ Foundation for commercial platform (if desired)

---

## 2. High-Level Goals & Non-Goals

### 2.1 Goals

**Architecture Correctness:**
- Consistent hexagonal + functional core across all repos
- Enforced via dependency-cruiser + ESLint + policy gates
- Pure logic layer (no I/O/time/env/randomness)
- Deterministic behavior for LLM agent safety

**Agent-First Design:**
- MCP skill catalog with deterministic, cacheable checks
- Contract-driven development (Zod, OpenAPI, Result types)
- Vector memory for incident retrieval and fix proposals
- Self-healing with guardrails (no silent mutations, policy validation)

**Universal Standards:**
- **Latest stable versions always:** All dependencies, tools, and runtimes use latest stable releases (pnpm 10.x, Node 24.x, React 19, Next 15.x, ESLint 9, TypeScript latest)
- Language-agnostic patterns (TS now, Rust-compatible)
- Policy-enforced dependencies (floors, bans, version constraints)
- File-type hygiene, TTL docs, SBOM generation
- Multi-language path (Python, Rust as specialized satellites)

**Self-Healing Ready:**
- Time-series telemetry (TimescaleDB)
- Vector embeddings for similarity search (pgvector + GPU)
- Agent-proposed fixes with risk scoring
- Autonomy levels from suggest-only to auto-remediation

### 2.2 Non-Goals (For Now)

- ❌ Public SaaS platform (foundation-first)
- ❌ Enforcing DevOS on every tiny script (repo tiers allow flexibility)
- ❌ Removing humans from high-risk decisions (architecture, security, payments)
- ❌ Multi-tenancy or public marketplace (internal-first, open-source later)

---

## 3. Architecture Overview (Hexagonal + Functional Core)

### 3.1 Folder Layout

**Philosophy:** Keep business logic absolutely clean and deterministic. Plug everything else via ports.

**Reference Implementation:** `packages/standards-cli/src/`

```
src/
  contracts/     # Data schemas (Zod, .d.ts, OpenAPI, Pydantic, Rust structs)
  logic/         # Domain logic: pure functions, no I/O/time/env/random
  ports/         # Interfaces for all I/O boundaries (DB, queues, AI, telemetry)
  services/      # Orchestration: use-cases composing logic + ports (DI-based)
  adapters/      # Concrete implementations (Postgres, HTTP, S3, LLMs)
  entrypoints/   # Edges: HTTP servers, CLIs, workers wiring services + adapters
tests/
  unit/          # Test logic + services with in-memory doubles
  integration/   # Test adapters with real infrastructure
  doubles/       # In-memory implementations (InMemoryRepo, FixedClock, SeqId)
policy/
  RULES.md
  constraints/   # Dependency floors, bans, version constraints
  skills/        # MCP skill manifests (YAML)
  sbom-policy.rego
```

**Folder Mapping Note:**
- `ports/` may be implemented as separate folder OR within `contracts/` (interface definitions)
- Current standards-cli: uses `contracts/` for interfaces + schemas
- Both patterns valid; key is **logical separation of concerns**

**Current Status:**
- ✅ Verified in `packages/standards-cli`
- ✅ Enforced via dependency-cruiser + ESLint
- ✅ 19 skills following this pattern

**See Also:**
- Detailed layer rules: [§3.2 Dependency Rules](#32-dependency-rules-who-can-import-whom)
- DI pattern: [§4.2 Dependency Injection](#42-dependency-injection-pattern-services)

### 3.2 Dependency Rules (Who Can Import Whom)

**Enforced via:** `packages/standards-cli/assets/dep-cruiser.config.cjs` (canonical)

**Summary of Rules:**

| Layer | Can Import | Cannot Import | Rationale |
|-------|------------|---------------|-----------|
| `contracts/` | Nothing | — | Pure types, used by all layers |
| `logic/` | `contracts/`, other `logic/` | `ports/`, `services/`, `adapters/`, I/O libs | **Purity:** deterministic, testable, agent-safe |
| `ports/` | `contracts/` | `logic/`, `services/`, `adapters/` | Interface definitions only |
| `services/` | `contracts/`, `logic/`, `ports/` | `adapters/` | **DI:** accepts adapters via constructor |
| `adapters/` | `contracts/`, `ports/`, external libs | `logic/` | I/O implementation, respects contracts |
| `entrypoints/` | `services/`, `adapters/`, `contracts/`, config | Direct `logic/` calls discouraged | Wiring layer only |

**Exceptions:** None currently. Historical violations must migrate incrementally.

**Validation:**
- Local: `pnpm check-fast` (runs dep-cruiser)
- CI: FAST gate fails on violations
- Config: `packages/standards-cli/assets/dep-cruiser.config.cjs`

**See Also:**
- Purity enforcement: [§4.1 Logic Purity](#41-logic-purity-srclogic)
- ESLint rules: Documented in standards-cli package

---

## 4. Purity & DI Rules

### 4.1 Logic Purity (`src/logic/**`)

**Philosophy:** Logic layer must be pure for agent safety and deterministic behavior.

**Allowed:**
- ✅ Pure functions (input → output, no side effects)
- ✅ Deterministic Math (`Math.abs`, `Math.floor`, `Math.max`)
- ✅ String/array manipulation
- ✅ `Result<T, E>` or discriminated unions for error handling
- ✅ `assertUnreachable(x: never)` for exhaustiveness

**Forbidden:**
- ❌ `Date.now()`, `Math.random()`, `crypto.randomUUID()`
- ❌ `process.env`, `process.cwd()`, `process.exit()`
- ❌ `fs`, `http`, `https`, `child_process`, database drivers
- ❌ `throw` for business errors (use `Result` types)
- ❌ Global singletons or mutable module state

**Enforcement:**
- ESLint rules forbid imports (standards-cli config)
- Dependency-cruiser blocks logic → adapters edges
- CI FAST gate fails on violations

**Example (Correct):**
```typescript
// Example (simplified for documentation)
// See packages/standards-cli/src/logic/ for exact implementation

// ✅ Pure logic
export function calculateOrderTotal(
  items: OrderItem[],
  taxRate: number
): Result<Money, OrderError> {
  if (items.length === 0) {
    return Err({ type: "EMPTY_ORDER" })
  }

  const subtotal = items.reduce((sum, item) =>
    sum + item.price * item.quantity, 0)
  const tax = subtotal * taxRate
  const total = subtotal + tax

  return Ok({ amount: total, currency: "USD" })
}
```

**Example (Incorrect - DO NOT DO THIS):**
```typescript
// ❌ Impure logic
export function calculateOrderTotal(items: OrderItem[]): Money {
  const now = new Date()  // ❌ Non-deterministic
  const taxRate = parseFloat(process.env.TAX_RATE)  // ❌ Environment

  if (isHoliday(now)) {  // ❌ Hidden I/O
    return applyDiscount(items)
  }

  throw new Error("Not a holiday")  // ❌ Throw for business logic
}
```

### 4.2 Dependency Injection Pattern (`services/`)

**Philosophy:** Services orchestrate logic + ports via DI for testability.

**Pattern:**
- Services are **factory functions**: `makeXService(deps) -> XService`
- `deps` is plain object of port interfaces
- No global singletons
- No direct adapter imports (passed in)

**Example:**
```typescript
// Example (simplified for documentation)
// See packages/standards-cli/src/services/ for exact implementation

// ports/ (or contracts/)
export interface Clock { nowISO(): string }
export interface IdGen { ulid(): string }
export interface OrderRepo {
  save(order: Order): Promise<Result<void, RepoError>>
  findById(id: string): Promise<Result<Order | null, RepoError>>
}

// services/OrderService.ts
export interface OrderService {
  create(input: OrderInput): Promise<Result<Order, OrderError>>
}

export function makeOrderService(deps: {
  clock: Clock
  id: IdGen
  repo: OrderRepo
}): OrderService {
  return {
    async create(input) {
      // 1. Call pure logic
      const validation = validateOrderInput(input)  // from logic/
      if (validation.isErr()) return validation

      // 2. Use injected ports
      const order: Order = {
        id: deps.id.ulid(),
        createdAt: deps.clock.nowISO(),
        ...input
      }

      // 3. Delegate I/O to port
      return deps.repo.save(order)
    }
  }
}
```

**Wiring (entrypoints):**
```typescript
import { makeOrderService } from "../services/OrderService"
import { PgOrderRepo } from "../adapters/PgOrderRepo"
import { SystemClock } from "../adapters/SystemClock"
import { UlidGen } from "../adapters/UlidGen"

const orderService = makeOrderService({
  clock: new SystemClock(),
  id: new UlidGen(),
  repo: new PgOrderRepo(pool)
})
```

**Testing (with doubles):**
```typescript
import { makeOrderService } from "../../src/services/OrderService"
import { InMemoryOrderRepo } from "../doubles/InMemoryOrderRepo"
import { FixedClock } from "../doubles/FixedClock"
import { SeqId } from "../doubles/SeqId"

test("creates order with sequential IDs and fixed time", async () => {
  const service = makeOrderService({
    clock: new FixedClock("2025-01-01T00:00:00Z"),
    id: new SeqId("order"),
    repo: new InMemoryOrderRepo()
  })

  const result = await service.create({ items: [/* ... */] })

  expect(result.unwrap()).toEqual({
    id: "order-1",
    createdAt: "2025-01-01T00:00:00Z",
    // ...
  })
})
```

### 4.3 Testing & Mocking Policy

**Philosophy:** Mocks hide bugs. Test boundaries with real or in-memory implementations.

**Rules:**
- ❌ No `vi.mock` / `jest.mock` for internal code
- ❌ No global mocks (Date, Math.random)
- ✅ Boundary-only mocking: External APIs (Stripe, HubSpot, OpenAI) via MSW/nock
- ✅ In-memory doubles for ports (in `tests/doubles/`)

**Test Doubles:**
```
tests/doubles/
  InMemoryOrderRepo.ts     # Implements OrderRepo with Map
  InMemoryEventBus.ts      # Implements EventBus with array
  FixedClock.ts            # Returns "2025-01-01T00:00:00Z"
  SeqId.ts                 # Returns "prefix-1", "prefix-2", ...
  FakeLLM.ts               # Returns canned responses
```

**External API Mocking (Allowed):**
```typescript
// ✅ Mock external HTTP with MSW
import { rest } from "msw"
import { setupServer } from "msw/node"

const server = setupServer(
  rest.post("https://api.stripe.com/v1/charges", (req, res, ctx) => {
    return res(ctx.json({ id: "ch_test", status: "succeeded" }))
  })
)
```

**Enforcement:**
- ESLint rule forbids `vi.mock` outside `tests/external/`
- Code review policy
- Documented in AGENTIC_IMPLEMENTATION_PLAN

---

## 5. Telemetry & Vector Memory (Time-Based DB + Embeddings)

### 5.1 Telemetry Model

**Port Definition:**
```typescript
export interface Telemetry {
  record(event: TelemetryEvent): Promise<void>
  query(filter: TelemetryQuery): Promise<TelemetryEvent[]>
}

export interface TelemetryEvent {
  id: string
  ts: string  // ISO 8601
  source: "ci" | "runtime" | "agent" | "user"
  kind: "error" | "warn" | "info"
  code?: string  // e.g., "TEST_FAILURE", "POLICY_VIOLATION"
  message: string
  traceId?: string
  tags: Record<string, string>
  payload?: unknown  // Redacted (see §5.4)
}
```

**Adapter Implementation (Current Reference):**
- ✅ **TimescaleDB** (hjps-ops/docker-compose.observability.yml)
- ✅ **@h-j-petroleum/toolkit-logger@0.0.6** with TimescaleTransport
- ✅ **End-to-end verified:** 41 logs with embeddings

**Schema:** Canonical in [observability/IMPLEMENTATION_PLAN.md](planning/observability/IMPLEMENTATION_PLAN.md#unified-database-schema)

### 5.2 Embeddings & Vector Store

**Port Definition:**
```typescript
export interface EmbeddingStore {
  embed(input: Embeddable): Promise<Embedding>
  upsert(record: EmbeddingRecord): Promise<void>
  search(query: EmbeddingQuery): Promise<EmbeddingSearchResult[]>
}

export interface Embeddable {
  text: string
  metadata?: Record<string, unknown>
}

export interface Embedding {
  vector: number[]  // 1024-dim for BGE-large-en-v1.5
  model: string
}
```

**Adapter Implementation (Current Reference):**
- ✅ **pgvector extension** (Postgres 16 + TimescaleDB)
- ✅ **GPU embeddings service** (FastAPI + ONNX Runtime + CUDA)
  - Service Name: `local-llm` (container: `hjps-ops-local-llm`)
  - Model: BAAI/bge-large-en-v1.5 (1024 dimensions)
  - Image: `nvidia/cuda:12.8.0-cudnn-runtime-ubuntu24.04` (public base)
  - Endpoint: `local-llm:8000` in hjps-ops stack
  - Batch support: up to 32 texts per request

- ✅ **Coding Agent Service** (vLLM)
  - Service Name: `vllm-chat` (container: `hjps-ops-vllm-chat`)
  - Model: `Qwen/Qwen2.5-32B-Instruct-AWQ`
  - Image: `vllm/vllm-openai:latest` (public hosted)
  - Endpoint: `vllm-chat:8000` (internal) / `19000` (host)
  - Capabilities: Chat, Code Completion, Tool Calling

**Use Cases:**
- Error similarity: "Find past incidents similar to this test failure"
- Fix retrieval: "Find PRs that fixed similar stack traces"
- Code search: "Find modules handling similar input validation"
- Spec search: "Find docs related to this API endpoint"

**Performance (Reference Implementation):**
- Embedding generation: ~50ms per text (GPU)
- Similarity search: <500ms for top 10 results

**Schema:** Canonical in [observability/IMPLEMENTATION_PLAN.md](planning/observability/IMPLEMENTATION_PLAN.md#unified-database-schema)

### 5.3 Database Schemas

**Canonical Location:** [observability/IMPLEMENTATION_PLAN.md](planning/observability/IMPLEMENTATION_PLAN.md#unified-database-schema)

**Overview (link to full schemas):**
- `application_logs`: General telemetry (with `message_embedding vector(1024)`)
- `test_runs`: Local vs CI test execution
- `test_failures`: Individual test failures (with `error_embedding vector(1024)`)
  - FK: `test_failures.run_id → test_runs.run_id ON DELETE CASCADE`
- `lint_failures`: Linter violations
  - FK: `lint_failures.run_id → test_runs.run_id ON DELETE CASCADE`
- `selfheal_proposals`: Fix proposals + outcomes (with `proposal_embedding vector(1024)`)
  - `trigger_id` format: documented as `run:{run_id}`, `error:{error_id}`, etc.
  - `pattern_id`: SHA256(error_message + stack_frames + affected_files)
- `trace_spans`: Distributed traces (OTEL)
- `metrics_samples`: Runtime metrics

**See:** [observability/IMPLEMENTATION_PLAN.md](planning/observability/IMPLEMENTATION_PLAN.md) for complete CREATE TABLE statements

### 5.4 Privacy & Secret Redaction

**Risk:** Telemetry may capture PII, tokens, passwords in error messages/stack traces.

**Redaction Strategy:**

1. **Pre-storage redaction** (in toolkit-logger transport):
   ```typescript
   // Example (simplified for documentation)
   function redactSecrets(text: string): string {
     return text
       .replace(/Bearer\s+[A-Za-z0-9_-]+/gi, "Bearer [REDACTED]")
       .replace(/api[_-]?key[:\s]+[A-Za-z0-9_-]+/gi, "api_key: [REDACTED]")
       .replace(/sk-[A-Za-z0-9]{20,}/g, "sk-[REDACTED]")
       .replace(/password[:\s]+\S+/gi, "password: [REDACTED]")
       .replace(/email[:\s]+\S+@\S+/gi, "email: [REDACTED]")
   }
   ```

2. **Allowlist for source code** (lint_failures.source):
   - Only store if file_path in `/src/`, `/tests/`, `/scripts/`
   - Exclude `.env`, `secrets.yml`, `credentials.json`

3. **Audit logging:**
   - All telemetry writes logged to separate audit table
   - Periodic review for PII leakage
   - Retention: 30 days audit logs

**Enforcement:**
- Unit tests validate redaction patterns
- Integration tests with known PII strings

---

## 6. Self-Healing & Failure Handling

### 6.1 SelfHealer Ports & Service

**Port Definitions:**
```typescript
// Example (simplified for documentation)
// See self-healing implementation docs for full types

export interface SelfHealer {
  proposeFix(signal: FailureSignal): Promise<FixProposal | null>
  applyFix(proposal: FixProposal): Promise<SelfHealOutcome>
}

export interface AgentPort {
  proposeFix(ctx: {
    signal: FailureSignal
    similarIncidents: EmbeddingSearchResult[]
    codeContext: CodeContext
  }): Promise<FixProposal>
}

export interface FailureSignal {
  id: string  // ulid
  type: "test_failure" | "ci_failure" | "runtime_error" | "policy_violation"
  timestamp: string
  patternId: string  // SHA256(error_message + stack_frames + affected_files)
  context: {
    repository: string
    branch: string
    commitSha: string
    errorMessage: string
    stackTrace?: string
    affectedFiles?: string[]
    runId?: string  // Links to test_runs.run_id
  }
}

export interface FixProposal {
  id: string  // ulid
  patternId: string  // Same as FailureSignal.patternId for SLO tracking
  description: string
  diffs: CodeDiff[]
  testsToRun: string[]
  riskScore: number  // 0-100 (calculated by logic layer)
  confidence: number  // 0-100 (from agent)
  references: string[]  // IDs of similar past proposals/PRs
  agentContext: {
    model: string  // e.g., "claude-sonnet-4" (provider-agnostic)
    reasoning: string
    alternatives: string[]
  }
}
```

**Pattern Identity (pattern_id):**
- **Purpose:** Track self-healing attempts per unique error pattern (SLO enforcement)
- **Calculation:**
  ```typescript
  // Example (simplified for documentation)
  function computePatternId(signal: FailureSignal): string {
    const canonicalError = signal.context.errorMessage
      .toLowerCase()
      .trim()
      .replace(/\s+/g, ' ')

    const canonicalStack = signal.stackTrace
      ?.split('\n')
      .slice(0, 5)  // Top 5 stack frames
      .map(line => line.trim())
      .filter(Boolean)
      .join('|') || ''

    const canonicalFiles = signal.context.affectedFiles
      ?.slice()
      .sort()  // Deterministic order
      .join(',') || ''

    const input = `${canonicalError}::${canonicalStack}::${canonicalFiles}`
    return crypto.createHash('sha256').update(input).digest('hex')
  }
  ```
- **Storage:** `selfheal_proposals.pattern_id` (VARCHAR(64))
- **Index:** `CREATE INDEX idx_selfheal_pattern ON selfheal_proposals(pattern_id, timestamp DESC);`
  - Note: Non-unique index (allows multiple attempts per pattern for SLO counting)
- **SLO Query:**
  ```sql
  SELECT COUNT(*)
  FROM selfheal_proposals
  WHERE pattern_id = $1
    AND timestamp > NOW() - INTERVAL '1 day'
    AND status IN ('proposed', 'pr_opened');
  -- Result < 3 = can attempt self-heal
  ```

**Attempts Tracking:**
- **Storage:** `selfheal_proposals` table (each row = one attempt)
- **Query Logic:** Count proposals with matching `pattern_id` in time window
- **Escalation:** If count ≥ `config.slo.maxAttemptsPerPattern`, escalate to human

**Provider-Agnostic Design:**
- `AgentPort` interface is abstract (not Claude-specific)
- Implementations: `ClaudeAgentAdapter`, `OpenAIAgentAdapter`, `LocalLLMAdapter`
- Model specified in `agentContext.model`
- Configuration via env or DI:
  ```typescript
  const agent = config.agent.provider === "claude"
    ? new ClaudeAgentAdapter(config.agent.claude)
    : new OpenAIAgentAdapter(config.agent.openai)
  ```

**Kill Switches & Configuration:**
```typescript
// Example (simplified for documentation)
export interface SelfHealConfig {
  enabled: boolean  // Global kill switch
  slo: {
    maxAttemptsPerPattern: number  // Default: 3
    timeWindow: string  // Default: "1 day"
  }
  autonomy: {
    level: 0 | 1 | 2 | 3  // See §7
    autoMergeThreshold: {
      confidence: number  // Default: 90
      riskScore: number  // Default: 20 (max)
    }
  }
  concurrency: {
    maxConcurrentProposals: number  // Default: 5
    maxConcurrentPRs: number  // Default: 3
  }
  pausedPatterns: string[]  // Pattern IDs to skip
  rules: ValidationRule[]  // Policy checks
}
```

**Per-Repo Configuration (shipped defaults):**
```yaml
# .devos/self-heal.yml (ship with self-healing DISABLED by default)
enabled: false  # Must opt-in explicitly
autonomy:
  level: 0  # Suggest-only (human review required)
slo:
  maxAttemptsPerPattern: 3
  timeWindow: "1 day"
concurrency:
  maxConcurrentProposals: 5
pausedPatterns: []

# Branch naming enforcement (documented)
# All self-heal branches MUST use selfheal/* prefix
# CI checks enforce this naming convention
```

**SelfHealingService Implementation:**

Key points (full implementation in self-healing docs):
- Idempotent: check `signal.id` before processing
- Uses `pattern_id` for SLO tracking
- Calls logic layer for validation (pure functions)
- Provider-agnostic: takes `AgentPort` interface

### 6.2 Self-Healing State Machine (Detect → Retrieve → Propose → Apply)

**4-Step Flow (Without Magical Thinking):**

```
┌─────────────────┐
│ Detect Failure  │ Record to telemetry
└────────┬────────┘
         │
         v
┌─────────────────┐
│ Retrieve        │ Vector search for similar incidents
│ Similar         │
│ Incidents       │
└────────┬────────┘
         │
         v
    ┌────────┐
    │ Check  │
    │ SLO    │
    └───┬────┘
        │
    ┌───┴────┐
    │        │
pattern_id   SLO
attempts < 3 exceeded
    │        │
    v        v
┌─────────┐ ┌──────────────┐
│ Propose │ │ Escalate to  │
│ Fix     │ │ Human        │
└────┬────┘ └──────────────┘
     │
     │ Agent generates proposal
     │
     v
┌─────────────┐
│ Validate    │
│ Proposal    │
└──────┬──────┘
       │
   ┌───┴───┐
   │       │
Risk +    Rejected
rules OK
   │       │
   v       v
┌─────┐  ┌────┐
│Apply│  │Log │
│to   │  │+   │
│Branc│  │Skip│
│h    │  └────┘
└──┬──┘
   │
   │ Run FAST + FULL gates
   │
   v
┌──────────┐
│ Checks   │
│ Pass?    │
└─────┬────┘
      │
  ┌───┴────┐
  │        │
 Yes       No
  │        │
  v        v
┌────┐  ┌──────────┐
│Open│  │Validation│
│PR  │  │Failed    │
└─┬──┘  └──────────┘
  │
  │ Autonomy Level Decision
  │
  v
┌──────────────┐
│ Level 0:     │
│ Human Review │
│              │
│ Level 1+:    │
│ Check Auto-  │
│ Merge?       │
└──────────────┘
```

**Step 1: Detect**
- Triggers: CI test failure, runtime error, policy violation, SLO breach
- Event captured in telemetry with `signal.id`, `signal.patternId`, `signal.type`, `signal.context`

**Step 2: Retrieve**
- Query telemetry for recent events
- Generate embedding of error message + stack trace
- Vector search for similar past incidents (threshold 0.7)
- Return top 10 results with similarity scores
- **Performance:** <500ms

**Step 3: Propose**
- **Check SLO:** Query `selfheal_proposals` for attempts with matching `pattern_id` in last 24h
  - If count ≥ 3: escalate to human (pattern needs architectural fix, not patch)
- Load code context (affected files, recent changes, module deps)
- Call agent with: signal + similar incidents + code context
- Agent returns `FixProposal` (diffs, tests, reasoning)
- Calculate risk score (logic layer): lines changed, files affected, coverage delta, module criticality
- Validate proposal (logic layer): no changes to `logic/` without tests, no new deps without approval, no security-sensitive files

**Step 4: Apply**
- Create branch: `selfheal/{proposal.id}`
- Apply diffs via Git adapter
- Run FAST gate (lint, typecheck, unit tests, dep-cruiser)
- Run FULL gate (coverage, SBOM, security scans)
- Open PR with: title, body (proposal + reasoning + validation), labels (`self-heal`, `risk-{low|medium|high}`)
- **Autonomy decision:** (See [§7 Autonomy Levels](#7-autonomy-levels-human-vs-no-human-in-loop))

**Observability on Self-Healing:**
- Telemetry events: `selfheal.started`, `selfheal.slo_exceeded`, `selfheal.proposed`, `selfheal.validation_passed`, `selfheal.validation_failed`, `selfheal.pr_opened`, `selfheal.merged`
- Metrics: MTTR with vs without self-heal, proposal acceptance rate, recurring pattern detection
- Grafana dashboard: [observability/IMPLEMENTATION_PLAN.md](planning/observability/IMPLEMENTATION_PLAN.md#grafana-dashboards)

**Current Status:**
- ⏳ Phase 7 (Observability Phases 4-6)
- Infrastructure ready (telemetry + vector DB)
- Schema designed (with FK constraints + pattern tracking)
- Service logic specified
- Adapter implementations pending

---

## 7. Autonomy Levels (Human vs No-Human-in-Loop)

**Philosophy:** Incremental trust - don't jump from manual to fully autonomous.

**Full Ladder:**

### Level 0 – Suggest-Only (Current Default)
- Agents propose fixes (self-heal proposals)
- Open PRs as drafts
- No auto-merge, no auto-deploy
- All fixes require human approval
- **Use Case:** All repos initially, pilot phase

### Level 1 – Auto-Fix in Dev/Beta
- Safe fixes to `dev`/`beta` branches:
  - Limited change scope (e.g., only `logic/` + `tests/`)
  - No new dependencies
  - All gates green
  - Risk score < 30
- No direct writes to `main`
- **Use Case:** Tier 1 repos after 2 weeks of Level 0 data

### Level 2 – Auto-Merge Low-Risk Fixes to Main
- Low-risk changes auto-merge:
  - Scoped + tests + canary healthy
  - Confidence > 90%, Risk < 20
  - Strict risk scoring
- **Use Case:** Tier 2 repos after 4 weeks of Level 1 data

### Level 3 – Auto-Remediation in Prod
- Auto-rollback + auto-apply known safe playbooks
- Still exclude: core security/auth/payment logic
- **Use Case:** Future, after extensive Level 2 validation

**Mapping to Repo Tiers:**
- **Tier 0 (Lightweight):** Not eligible for self-healing (no telemetry/vector DB)
- **Tier 1 (Standard):** Eligible for Level 0-1
- **Tier 2 (Critical):** Eligible for Level 0-2 (Level 3 future consideration)

**Configuration:**
- Per-repo: `.devos/self-heal.yml#autonomy.level`
- Global defaults: standards repo config
- Override mechanism: repo-specific config takes precedence

---

## 8. Repo Tiers (How "Heavy" the Architecture Is)

**Philosophy:** Not every repo needs full DevOS. Choose tier based on criticality.

### Tier 0 – Lightweight / Utility

**Use Case:** Small tools, scripts, prototypes

**Requirements:**
- ✅ Basic lint, typecheck, some tests
- ✅ May not fully adopt hexagonal layers
- ❌ Telemetry/embeddings optional (not required)
- ❌ Self-healing not supported

**Examples:** CLI utilities, one-off scripts, experimental prototypes

### Tier 1 – Standard DevOS

**Use Case:** Typical business/production services (default for most)

**Requirements:**
- ✅ Full layers: contracts, logic, ports, services, adapters, entrypoints, tests
- ✅ `check-fast` + `check-full` required locally and in CI
- ✅ Dependency policy (floors, bans)
- ✅ Telemetry for critical flows (errors, slow queries)
- ⏳ Vector memory optional
- ⏳ Self-healing Level 0-1 (suggest or auto-fix dev/beta)

**Examples:** hjps-frontend, business APIs, internal tools

### Tier 2 – Critical / Self-Heal-Enabled

**Use Case:** Core backend systems, orchestrators, high-impact services

**Requirements:**
- ✅ Everything from Tier 1, plus:
- ✅ Full telemetry coverage (all flows logged)
- ✅ Vector memory (embeddings for errors, code, specs)
- ✅ Self-healing service configured
- ✅ Autonomy Level 0-2 (suggest → auto-fix → auto-merge low-risk)
- ✅ Stronger test thresholds (90%+ coverage)
- ✅ Policy gates (SBOM + OPA + security scans)

**Examples:** toolkit-standards, hjps-toolkit, hjps-ops (observability platform)

**Current Repo Tiers (Reference):**
- **toolkit-standards:** Tier 2 (houses standards-cli, enforces policy)
- **hjps-toolkit:** Tier 2 (core infrastructure packages)
- **hjps-ops:** Tier 2 (observability platform host)
- **hjps-frontend:** Tier 1 (business UI)
- **hubspot-dataops:** Tier 1 (planned pilot)

---

## 9. Enforcement Tooling (Local & CI)

### 9.1 Static Rules

**Dependency Rules:**
- **Tool:** dependency-cruiser
- **Config:** `packages/standards-cli/assets/dep-cruiser.config.cjs` (canonical)
- **Enforces:** logic ↔ adapters edges, service → adapter edges, etc.

**Linting:**
- **Tool:** ESLint (flat config)
- **Config:** standards-cli package
- **Enforces:** Forbid time/env/IO in `logic/`, no `vi.mock` in unit tests, etc.

**Type Checking:**
- **Tool:** TypeScript strict mode
- **Config:** `tsconfig.json` with `strict: true`, `noImplicitAny: true`

### 9.2 Dynamic Gates

**FAST Gate (≤ 2 min target):**
- `prettier-check` (changed files only)
- `eslint-check` (changed files only)
- `typecheck` (full project)
- Unit tests (vitest)
- `dep-cruiser` (layering rules)
- `script-map` validation

**FULL Gate (≤ 7 min target):**
- Everything in FAST, plus:
- Coverage thresholds (80%+ for Tier 2)
- SBOM generation + OPA policy (license/registry checks)
- File-type compliance
- Dependency constraints (floors, bans, overrides from `policy/node/constraints.json`)
- Security scans (vulnerability, secret)
- Build validation

**Current Implementation (Reference):**
- ✅ Scripts: `packages/standards-cli/src/cli/commands/check.ts`
- ✅ CI: `.github/workflows/policy-checks.yml`
- ✅ Skills: 19 defined in `mcp/skills/*.yml`

**See:** [POLICY_REFERENCE.md](POLICY_REFERENCE.md#skills-catalog) for skill details

---

## 10. Standards / Policy Repo Integration

### 10.1 Latest Stable Versions Policy

**Principle:** ALL development work MUST use latest stable versions of dependencies, tools, and runtimes.

**Philosophy:**
- Security: Latest versions include critical security patches
- Features: Access to newest capabilities and improvements
- Agent compatibility: Latest versions have better LLM training data
- Technical debt: Avoiding version sprawl and upgrade debt
- Performance: Latest versions often include speed improvements

**Enforced Versions (Current Reference - as of 2025-11-24):**
- **Package Manager:** pnpm 10.x (latest stable: 10.23.0)
- **Runtime:** Node 24.x (LTS)
- **TypeScript:** Latest stable (≥5.6)
- **Frameworks:**
  - React 19.x (latest)
  - Next.js 15.x (latest)
  - Vitest (latest)
- **Tooling:**
  - ESLint 9.x (flat config)
  - Prettier (latest)
  - Dependency-cruiser (latest)

**Validation:**
- `repo:workspace-check` skill enforces pnpm 10.x
- `deps:policy-node` skill validates framework floors
- CI gates block outdated major versions
- Config: `policy/node/constraints.json` (canonical source)

**Exceptions:**
- Explicit documented exceptions only (with expiration dates)
- Must have business justification (e.g., upstream library incompatibility)
- Require ADR for major version pins

**See:** `policy/node/constraints.json` for complete floors/bans/overrides

### 10.2 Mechanism

**Distribution:** NPM packages via GitHub Packages

**Published Packages (Current Reference - Phase 6 Week 2 Complete):**
- `@h-j-petroleum/toolkit-cli@0.0.3`
- `@h-j-petroleum/toolkit-config@0.0.2`
- `@h-j-petroleum/toolkit-logger@0.0.6` (with TimescaleTransport + structured logging)
- `@h-j-petroleum/toolkit-telemetry@0.0.10` (with auto-backfill + CI tracking)
- `@h-j-petroleum/toolkit-pdf@0.0.2-alpha.3`
- + 8 more packages

**Installation:**
```bash
pnpm add -D @h-j-petroleum/standards-cli
npx hjps-standards policy install
```

**Contents:**
- Shared ESLint/TS configs
- Shared dep-cruiser config
- Shared OPA policies
- Skill definitions (MCP YAML manifests)
- CLI helpers (`policy-install`, `policy-verify`, `check`, `check-full`)

**See:** [PHASE6_IMPLEMENTATION_PLAN.md](planning/PHASE6_IMPLEMENTATION_PLAN.md#workstream-3-hjps-toolkit-publisher-readiness--incremental-ts-migration--complete)

---

## 11. Implementation Roadmap

### 11.1 Current State

**Infrastructure:**
- ✅ Folder layout verified (standards-cli)
- ✅ Tests: Boundaries-only mocking enforced
- ✅ Telemetry: TimescaleDB + pgvector operational (Phases 1-2)
- ✅ Embeddings: GPU service + toolkit-logger verified (41 logs with embeddings)
- ✅ CI: policy-checks.yml with FAST/FULL gates
- ⏳ Self-healing: Phase 7 planned

**Phase Alignment:**

| Observability Phase | DevOS Phase | DevOS Week | Status |
|---------------------|-------------|------------|--------|
| Phase 1: Infrastructure | Phase 6 | Week 2 | ✅ Complete |
| Phase 2: Vector Integration | Phase 6 | Week 2 | ✅ Complete |
| Phase 3: Services | Phase 6 | Week 3 | ⏳ In Progress (Advanced) |
| Phase 4-6: Self-Healing | **Phase 7** | Weeks 5-7 | ⏳ Planned |

**Rationale for Phase 7 Separation:**
- Phase 6 = 4 weeks (aligned with current sprint)
- Self-healing = 3 additional weeks (test schemas, semantic search, agent access, apply/monitor)
- Explicit separation prevents scope creep
- Allows Phase 6 to complete with clear acceptance criteria

### 11.2 Refactor Plan (Incremental)

**Completed:**
- [x] Align structure (standards-cli implements pattern)
- [x] Introduce ports & DI (services use factory pattern)
- [x] Enforce rules (dep-cruiser + ESLint active)
- [x] Wire telemetry + embeddings (Phases 1-2 complete)
- [x] Infrastructure: `local-llm` and `vllm-chat` services operational

**Remaining:**
- [ ] Complete Phase 3 (services + frontend rollout)
- [ ] Introduce self-healing path (Phase 7)
  - [/] Test schemas (test_runs, test_failures, lint_failures) - *Infrastructure Added*
  - [ ] Semantic search (vector similarity for incidents)
  - [ ] MCP server (agent access tools)
  - [/] SelfHealingService (detect → retrieve → propose → apply) - *Partially In Progress*
  - [ ] Worker/daemon (subscribe to failure signals)
- [ ] Document & freeze (this canvas!)

**Timeline:**
- **Phase 6:** 4 weeks (Weeks 1-4)
- **Phase 7:** 3 weeks (Weeks 5-7)
- **Total:** 7 weeks from Phase 6 start

**See:**
- Phase 6 details: [PHASE6_IMPLEMENTATION_PLAN.md](planning/PHASE6_IMPLEMENTATION_PLAN.md)
- Phase 6 tasks: [PHASE6_TASKS.md](planning/PHASE6_TASKS.md)
- Observability plan: [observability/IMPLEMENTATION_PLAN.md](planning/observability/IMPLEMENTATION_PLAN.md)

---

## 12. Open Questions & Risks

### 12.1 Resolved

- ✅ Validation boundaries: Zod in contracts/, domain validation in logic/
- ✅ Foreign keys: Added with CASCADE for cleanup
- ✅ Privacy: Redaction strategy documented ([§5.4](#54-privacy--secret-redaction))
- ✅ Provider coupling: AgentPort is abstract, Claude is one adapter
- ✅ Pattern tracking: SHA256(error + stack + files) stored in pattern_id

### 12.2 Remaining

**Language Strategy:**
- ⏳ Rust satellite timeline (performance-critical components)
- ⏳ Python integration pattern (ML/analytics services)

**Public Release:**
- ⏳ Framework name (pending decision)
- ⏳ CLI bootstrap (`npx create-devos new-service`)
- ⏳ Public documentation site (devos.dev?)

**Commercial:**
- ⏳ DevOS Cloud pricing model (if/when Phase 2+)
- ⏳ Open-source vs proprietary split

### 12.3 Risks (Honest Assessment)

**Complexity & Ceremony:**
- DevOS is heavy for small tasks
- Must consciously choose tiers
- **Mitigation:** Tier 0 for lightweight use, explicit tier assignments

**Port Explosion:**
- Over-abstracting everything slows development
- **Mitigation:** Only create ports for boundaries that change (DB, AI, external APIs)

**Infra Burden:**
- Time-series DB + vector DB + policies + CI = more moving parts
- **Mitigation:** Hosted platform (DevOS Cloud) for teams that don't want to self-host

**False Confidence:**
- Green CI ≠ correct domain behavior
- **Mitigation:** Pure logic layer + property testing + canary deployments

**Bus Factor:**
- Mental model lives largely in one head
- **Mitigation:** This canvas + examples + onboarding docs

**Assumptions:**
- Rails are practical, not religious
- Investment in docs and examples
- Playgrounds/spikes where rules are lighter

---

## Cross-References

**Planning Documents:**
- Phase 6 implementation: [PHASE6_IMPLEMENTATION_PLAN.md](planning/PHASE6_IMPLEMENTATION_PLAN.md#workstream-4-cost--observability-enforcement-advisorywarn--in-progress)
- Phase 6 tasks: [PHASE6_TASKS.md](planning/PHASE6_TASKS.md#week-2--registry-publishing-changesets-perf--openapi--complete)
- Observability platform: [observability/IMPLEMENTATION_PLAN.md](planning/observability/IMPLEMENTATION_PLAN.md#implementation-phases)

**Architecture Documents:**
- Agentic patterns: [AGENTIC_IMPLEMENTATION_PLAN.md](../../agents/AGENTIC_IMPLEMENTATION_PLAN.md#architecture-principles)
- Policy reference: [POLICY_REFERENCE.md](POLICY_REFERENCE.md#skills-catalog)

**Policy Files (Canonical):**
- Dependency rules: `packages/standards-cli/assets/dep-cruiser.config.cjs`
- Dependency constraints: `policy/node/constraints.json`
- SBOM policy: `policy/sbom-policy.rego`

**Implementation:**
- Standards CLI: `packages/standards-cli/`
- Skills catalog: `mcp/skills/` (19 skills)
- Toolkit packages: `adopt_hjps_toolkit/packages/`

---

## 13. Project Management (Shape Up for AI)

### 13.1 Methodology
**Canonical Doc:** [Shape Up for AI Teams](../../project-management/shape-up-for-ai-teams.md)

**Philosophy:** Traditional Agile (Scrum/Kanban) is too ceremony-heavy for AI-augmented teams. We use a cycle-based, appetite-driven approach.

**Core Principles:**
1.  **Cycles (2 Weeks):** Fixed time boxes for focused work.
2.  **Appetite (Not Estimates):** "This is worth 2 days" vs "This will take 2 days".
3.  **Betting:** Explicitly choosing what to work on each cycle.
4.  **Cool-Down (2 Days):** Retrospective, docs, and shaping for next cycle.

### 13.2 Issue Templates
We use specialized GitHub Issue templates to drive this workflow:

- **[BET] Shape Up Bet (`bet.yml`)**: For shaped work ready for a cycle.
  - Fields: Appetite (Small/Medium/Large), Scope, AI-Readiness Checklist.
- **[IDEA] Idea / Feature (`idea.yml`)**: For the Inbox (unshaped).
- **[BUG] Bug Report (`bug_report.yml`)**: AI-optimized bug reporting.

### 13.3 Workflow
1.  **Inbox**: Ideas come in via `idea.yml`.
2.  **Shaping**: Humans refine ideas into Bets (defining appetite and boundaries).
3.  **Betting Table**: At cycle start, select Bets to fill the 2-week appetite.
4.  **Building**: AI agents execute Bets (using `bet.yml` context).
5.  **Review**: Humans verify against acceptance criteria.

---

## Appendix: Vision Document

For the full strategic context, see: [AI_NATIVE_DEVOS_VISION.md](devos-vision.md)

**Document Relationship:**
- **Vision Document:** Strategic context (the "why" and long-term direction)
- **This Canvas:** Operational patterns (the "how" and current implementation)

Key quotes from vision:

> "This is not just 'a framework' or 'some scripts'; it is the *operating system* for how all code in your universe is structured, tested, observed, reasoned about, and modified (by humans or agents)."

> "The long-term north star: A world where the majority of routine coding, bugfixing, and maintenance is performed by agents running on a well-structured, observable, policy-enforced architecture—and your role is to set direction, model domains, and make high-level tradeoffs."

**Commercial Path:**
- Near-term: AI-native dev studio (internal leverage)
- Long-term: DevOS Cloud (multi-tenant platform)

---

*End of Canvas*
