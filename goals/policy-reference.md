# Policy Reference

**Last Updated:** 2025-11-11  
**Policy-Version:** 1.1.0 (Phase 6)  
**TTL:** 90d  
**Owner:** Architecture + Security

---

## Overview

This document provides complete reference documentation for the policy-as-code system, including all skills, enforcement modes, and configuration options.

---

## Table of Contents

1. [Enforcement Modes](#enforcement-modes)
2. [Skills Catalog](#skills-catalog)
3. [Exception Management](#exception-management)
4. [Metrics and Monitoring](#metrics-and-monitoring)
5. [Configuration Files](#configuration-files)

---

## Enforcement Modes

The policy system supports three enforcement modes:

### Advisory Mode

- **Purpose**: Data collection phase
- **Behavior**: Collects metrics, reports findings, does not block PRs
- **Use Case**: Initial adoption, baseline data collection
- **Duration**: Typically 1-2 weeks

### Warn Mode

- **Purpose**: Warning phase
- **Behavior**: Reports findings, generates warnings, does not block PRs
- **Use Case**: Transition period before full enforcement
- **Duration**: Typically 1-2 weeks
- **Rollback Criteria**: Failure rate > 10%, Red PR rate > 15%

### Enforce Mode

- **Purpose**: Full enforcement
- **Behavior**: Blocks PRs with policy violations
- **Use Case**: Production enforcement
- **Rollback Criteria**: Failure rate > 5%, Red PR rate > 10%

---

## Skills Catalog

### Style Skills

#### `style:prettier-check`

- **Description**: Validates code formatting using Prettier
- **Gate**: FAST
- **Default Mode**: warn
- **Dependencies**: prettier

#### `style:eslint-check`

- **Description**: Validates code quality using ESLint
- **Gate**: FAST
- **Default Mode**: warn
- **Dependencies**: eslint

#### `style:jsdoc-coverage`

- **Description**: Enforces JSDoc documentation coverage
- **Gate**: FULL
- **Default Mode**: warn
- **Threshold**: 70% minimum coverage

### Architecture Skills

#### `arch:dep-cruise`

- **Description**: Validates dependency architecture and layer boundaries
- **Gate**: FULL
- **Default Mode**: enforce
- **Dependencies**: dependency-cruiser

#### `arch:module-system-check` (Phase 6)

- **Description**: Validates per-package module type (CJS/ESM) consistency and TypeScript NodeNext compatibility
- **Gate**: FULL
- **Default Mode**: enforce
- **Artifact**: `artifacts/arch/modules.json`
- **Runtime**: ≤ 3s

#### `logic:purity-scan`

- **Description**: Scans logic layer for impure functions
- **Gate**: FULL
- **Default Mode**: enforce
- **Dependencies**: eslint

### Dependency Skills

#### `deps:policy-node`

- **Description**: Validates Node.js dependency constraints
- **Gate**: FULL
- **Default Mode**: warn
- **Checks**: Banned packages, license compliance, duplicate majors, packageManager, engines.node

#### `deps:file-protocol-check` (Phase 6)

- **Description**: Scans for file: protocol dependencies and reports migration guidance
- **Gate**: FAST (advisory), FULL (advisory → enforce)
- **Default Mode**: advisory
- **Artifact**: `artifacts/deps/file-deps.json`
- **Runtime**: ≤ 2s

#### `deps:publish-check` (Phase 6)

- **Description**: Validates publisher readiness for GitHub Packages (.npmrc, publishConfig, workflow)
- **Gate**: FULL
- **Default Mode**: advisory
- **Artifact**: `artifacts/deps/publish.json`
- **Runtime**: ≤ 2s

#### `deps:policy-python` (Phase 6)

- **Description**: Validates Python dependency constraints (pyproject.toml, lock files)
- **Gate**: FULL (advisory)
- **Default Mode**: advisory
- **Artifact**: `artifacts/python/deps.json`
- **Runtime**: ≤ 5s

### Repository Skills

#### `repo:script-map`

- **Description**: Maps available package.json scripts
- **Gate**: FAST
- **Default Mode**: advisory

#### `repo:hygiene`

- **Description**: Validates repository hygiene (gitignore, editorconfig, etc.)
- **Gate**: FULL
- **Default Mode**: warn

#### `repo:workspace-check` (Phase 6)

- **Description**: Detects workspace manager and enforces pnpm@10.x (latest stable) and Node ≥24
- **Gate**: FULL
- **Default Mode**: advisory → enforce
- **Artifact**: `artifacts/repo/workspace.json`
- **Runtime**: ≤ 2s
- **DevOS Principle**: Latest stable versions only (see [DevOS Canvas §10.1](DEVOS_ARCHITECTURE_CANVAS.md#101-latest-stable-versions-policy))

#### `repo:language-check` (Phase 6)

- **Description**: Detects primary languages (Node/Python) and drives skill selection
- **Gate**: FULL
- **Default Mode**: advisory
- **Artifact**: `artifacts/repo/languages.json`
- **Runtime**: ≤ 2s

### Documentation Skills

#### `docs:balance`

- **Description**: Checks documentation-to-code ratio
- **Gate**: FULL
- **Default Mode**: warn

#### `docs:freshness`

- **Description**: Validates documentation TTL compliance
- **Gate**: FULL
- **Default Mode**: advisory

### Security Skills

#### `security:vulnerability-scan`

- **Description**: Scans dependencies for known CVEs
- **Gate**: FAST
- **Default Mode**: enforce
- **Dependencies**: pnpm audit

#### `security:secret-scan`

- **Description**: Detects hardcoded secrets
- **Gate**: FULL
- **Default Mode**: warn
- **Dependencies**: gitleaks (optional)

### Quality Skills

#### `test:coverage`

- **Description**: Enforces minimum test coverage thresholds
- **Gate**: FULL
- **Default Mode**: warn
- **Threshold**: 80% minimum coverage

#### `build:validation`

- **Description**: Verifies project builds successfully
- **Gate**: FULL (moved from FAST in Phase 6, change-aware in FAST)
- **Default Mode**: enforce
- **Dependencies**: build script in package.json

### SBOM Skills

#### `sbom:generate`

- **Description**: Generates Software Bill of Materials
- **Gate**: FULL
- **Default Mode**: warn
- **Dependencies**: cdxgen (optional)

#### `sbom:policy-eval`

- **Description**: Evaluates SBOM against policy using OPA
- **Gate**: FULL
- **Default Mode**: enforce
- **Dependencies**: opa (optional)

### File Type Skills

#### `file:type-compliance`

- **Description**: Validates file types using magika
- **Gate**: FULL
- **Default Mode**: warn
- **Dependencies**: magika (optional)

### Performance Skills (Phase 6)

#### `perf:bundle-budget-check`

- **Description**: Validates bundle sizes against performance budgets (25 KB gz default, 150 KB initial route)
- **Gate**: FULL
- **Default Mode**: enforce
- **Artifact**: `artifacts/perf/bundle.json`
- **Runtime**: ≤ 5s
- **Dependencies**: Bundle analysis reports in `.next/analyze/`

#### `perf:lighthouse-check`

- **Description**: Runs Lighthouse CI and validates Performance ≥90 and Accessibility ≥90
- **Gate**: FULL (FULL-only, not in FAST)
- **Default Mode**: enforce
- **Artifact**: `artifacts/perf/lhci.json`
- **Runtime**: 60-120s
- **Dependencies**: Lighthouse CI
- **Status**: CI-only, MVP implementation. Currently uses placeholder parsing; requires LHCI artifacts from CI pipeline. See skill implementation for setup requirements.

#### `perf:api-latency`

- **Description**: Validates API p95 latency against budgets (reads < 250ms, writes < 500ms)
- **Gate**: FULL
- **Default Mode**: enforce
- **Artifact**: `artifacts/perf/api-latency.json`
- **Runtime**: ≤ 5s
- **Status**: Advisory until data sources are wired. Expects `artifacts/perf/api-latency.json` or fixtures. Will be upgraded to enforce mode once API template emits traces/fixtures.

### API Skills (Phase 6)

#### `api:openapi-check`

- **Description**: Validates OpenAPI schema and ensures codegen is diff-clean
- **Gate**: FULL
- **Default Mode**: enforce
- **Artifact**: `artifacts/api/openapi.json`
- **Runtime**: ≤ 10s

### Observability Skills (Phase 6)

#### `obs:otel-check`

- **Description**: Validates required OTel attributes and traceparent propagation
- **Gate**: FULL
- **Default Mode**: advisory → warn
- **Artifact**: `artifacts/obs/otel.json`
- **Runtime**: ≤ 3s
- **Required Attributes**: trace_id, request_id, user_id, pipeline_id, batch_id, duration_ms

### Data Skills (Phase 6)

#### `data:migration-check`

- **Description**: Validates DB migration checksums and detects drift
- **Gate**: FULL (advisory)
- **Default Mode**: advisory
- **Artifact**: `artifacts/data/migrations.json`
- **Runtime**: ≤ 5s
- **Supports**: Alembic, Flyway

---

## Exception Management

### Exception Structure

```json
{
  "id": "exception-001",
  "skill": "security:vulnerability-scan",
  "package": "lodash@4.17.20",
  "file": "package.json",
  "reason": "Known false positive - CVE not applicable",
  "grantedBy": "security-team",
  "grantedDate": "2025-01-10",
  "expires": "2025-04-10",
  "adr": "docs/adr/2025-01-10-lodash-cve-exception.md",
  "status": "active"
}
```

### Exception Fields

- **id**: Unique identifier for the exception
- **skill**: Skill ID that the exception applies to
- **package**: Package name (optional, for dependency-related exceptions)
- **file**: File path (optional, for file-specific exceptions)
- **reason**: Justification for the exception
- **grantedBy**: Team or person who granted the exception
- **grantedDate**: Date when exception was granted (YYYY-MM-DD)
- **expires**: Expiration date (YYYY-MM-DD)
- **adr**: Path to ADR documenting the exception (optional)
- **status**: active, expired, or revoked

### Creating Exceptions

1. Create an ADR documenting the exception
2. Add exception to `policy/exceptions.json`
3. Update metadata (lastUpdated, counts)
4. Commit and push changes

### Exception Expiration

Exceptions automatically expire on their expiration date. The system warns about exceptions expiring within 30 days.

---

## Metrics and Monitoring

### Metrics Collection

Metrics are automatically collected and stored in `artifacts/metrics.json`:

```json
{
  "entries": [
    {
      "gateName": "fast",
      "executionTime": 45000,
      "timestamp": "2025-01-20T10:00:00Z",
      "skills": [
        {
          "skillId": "style:prettier-check",
          "success": true,
          "durationMs": 5000,
          "findingCount": 0,
          "errorCount": 0
        }
      ],
      "overallSuccess": true,
      "failureRate": 0.0
    }
  ],
  "lastUpdated": "2025-01-20T10:00:00Z"
}
```

### Key Metrics

- **Overall Failure Rate**: Percentage of gate runs that fail
- **Skill Failure Rates**: Failure rate per skill
- **PR Failure Rate**: Percentage of PRs blocked by policy
- **Execution Time**: Time taken for gate execution
- **Trend Analysis**: Improving, stable, or worsening trends

### Monitoring Dashboard

Use `scripts/metrics-dashboard.ts` to generate human-readable metrics with visualizations:

```bash
tsx scripts/metrics-dashboard.ts
```

The dashboard includes:
- ASCII charts for failure rate trends
- Skill comparison bar charts
- Execution time visualizations
- Advanced analytics (moving averages, volatility, correlations)

### Metrics Export

Export metrics to external monitoring systems:

```bash
# Export to Prometheus format
tsx scripts/export-metrics.ts --format=prometheus --output=metrics.prom

# Export to JSON format
tsx scripts/export-metrics.ts --format=json --output=metrics.json
```

### Exception Cleanup

Automatically mark expired exceptions:

```bash
# Dry run (preview changes)
tsx scripts/cleanup-exceptions.ts --dry-run

# Apply cleanup
tsx scripts/cleanup-exceptions.ts
```

---

## Configuration Files

### enforcement-mode.json

Defines enforcement mode and transition schedule:

```json
{
  "mode": "advisory",
  "transitionSchedule": {
    "advisory": {
      "startDate": "2025-01-01",
      "endDate": "2025-01-14"
    },
    "warn": {
      "startDate": "2025-01-15",
      "endDate": "2025-01-28",
      "rollbackCriteria": {
        "failureRateThreshold": 0.10,
        "redPRThreshold": 0.15
      }
    }
  },
  "skills": {
    "arch:dep-cruise": "enforce"
  },
  "monitoring": {
    "enabled": true,
    "metricsCollection": true,
    "failureLogPath": "artifacts/failures.json"
  }
}
```

### gates.json

Defines gate compositions:

```json
{
  "gates": {
    "fast": {
      "name": "FAST Gate",
      "description": "Quick checks for agent loop (≤2 min)",
      "skills": [
        "style:prettier-check",
        "style:eslint-check",
        "typecheck"
      ],
      "timeout": 120000
    },
    "full": {
      "name": "FULL Gate",
      "description": "Comprehensive checks for pre-PR",
      "skills": ["*"],
      "timeout": 300000
    }
  },
  "defaultGate": "fast"
}
```

### exceptions.json

Manages policy exceptions:

```json
{
  "exceptions": [],
  "metadata": {
    "lastUpdated": "2025-01-20",
    "totalExceptions": 0,
    "activeExceptions": 0,
    "expiredExceptions": 0
  }
}
```

---

## Best Practices

1. **Start with Advisory**: Always begin with advisory mode
2. **Monitor Metrics**: Regularly review failure rates and trends
3. **Document Exceptions**: Always create ADRs for exceptions
4. **Gradual Rollout**: Transition enforcement modes gradually
5. **Team Communication**: Keep team informed about policy changes

---

## Support

For questions or issues:

1. Review this reference documentation
2. Check [Adoption Guide](../../guides/ADOPTION_GUIDE.md) for usage instructions
3. Review [Migration Guide](../../guides/MIGRATION_GUIDE.md) for migration steps
4. Create an issue in `toolkit-standards` repository

