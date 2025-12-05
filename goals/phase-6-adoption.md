# Phase 6 Implementation Plan — Agentic Standards Adoption

Last updated: 2025-12-03
Policy-Version: 1.0.0
TTL: 90d
Owner: Architecture + Security
Status: In Progress (Workstreams 3 & 4 Partially Complete)

---

## Executive Summary

**Architecture Framework:** This implementation follows [AI-Native DevOS Architecture Canvas](../DEVOS_ARCHITECTURE_CANVAS.md)—an architecture + policy framework that acts as a development operating system for building systems when most code is touched by AI agents.

This plan operationalizes Phase 6 with enforceable, low-latency guardrails and a clear adoption path across repos. It converts documented performance, cost, and observability standards into executable skills and updates gates for speed and consistency. It also standardizes on pnpm 10.x (latest stable), migrates internal dependencies to GitHub Packages, and begins an incremental TypeScript (+Changesets) migration for hjps-toolkit.

**Strategic Update (Phase 6.1):** All new and existing skills will be packaged into a unified CLI tool, `@h-j-petroleum/standards-cli`, to improve developer experience, versioning, and performance. This replaces the previous `scripts/skills/*.ts` approach.

**Status Update (2025-11-24):**

✅ **Workstream 3 (hjps-toolkit Publisher Readiness) — COMPLETE**
- All 13 packages migrated to TypeScript with ESM
- Changesets fully integrated with automated beta and stable release workflows
- Multiple packages published to GitHub Packages under `@h-j-petroleum` scope (toolkit-cli@0.0.3, toolkit-config@0.0.2, toolkit-logger@0.0.6, toolkit-telemetry@0.0.10, toolkit-pdf@0.0.2-alpha.3, and 8 others)
- Security fixes: 4 Dependabot + 30 CodeQL alerts resolved
- New features: undici instrumentation, TimescaleDB transport with vector embeddings, PDF TypeScript conversion

✅ **Workstream 4 (Observability Infrastructure) — Phases 1-2 COMPLETE**
- Phase 1: Infrastructure deployed (TimescaleDB + pgvector, Grafana, OTEL Collector, custom GPU embeddings service)
- Phase 2: Vector integration complete (@h-j-petroleum/toolkit-logger@0.0.6 with TimescaleTransport, logs with embeddings verified across hjps-ops services)
- Phase 3: In Progress (hjps-ops services updated, end-to-end log visibility verified)
- Model: BAAI/bge-large-en-v1.5 (1024 dimensions) via FastAPI + ONNX Runtime + CUDA
- Embeddings service: Local OpenAI-compatible service at `http://embeddings-1:8000` (not direct OpenAI API)

🚧 **Workstream 5 (Type Safety Refactor) — IN PROGRESS**
- **Scope:** Comprehensive type safety improvements across entire codebase (83 files with unsafe assertions, 24 files with `any` types, 30+ files with weak index signatures)
- **Strategy:** Zod-based runtime validation at IO boundaries + realistic HubSpot data modeling + branded types
- **Timeline:** Weeks 2-4 (aligned with toolkit → ops → dataops rollout)
- **Week 2:** Foundation (toolkit-validation package with Zod schemas for all HubSpot objects, validation helpers: toNumber, sanitizeForShell)
- **Week 3:** Toolkits (toolkit-hubspot schema validation, JSON.parse security, environment config hardening)
- **Week 4:** DataOps + Standards (hubspot-dataops services, PostgreSQL validation, validation:schema-check skill)
- **Security Impact:** Command injection vulnerabilities fixed (environment validator shell sanitization), input validation at all boundaries
- **Observability Integration:** All validation failures logged to TimescaleDB with vector embeddings (enables Phase 7 self-healing: detect → retrieve → propose → apply)
- **Documentation:** [ADR-001](../../adr/ADR-001-ZOD-RUNTIME-VALIDATION.md), [ADR-002](../../adr/ADR-002-HUBSPOT-DATA-MODELING.md), [ADR-003](../../adr/ADR-003-VALIDATION-TESTING-STRATEGY.md), [Migration Guide](../../guides/TYPE_SAFETY_MIGRATION_GUIDE.md)

🚧 **Next Steps:**
- Implement remaining policy skills (perf:*, api:*, obs:otel-check, repo:*, deps:*)
- Complete Phase 3 observability rollout (frontend telemetry, initial Grafana dashboards)
- Pilot integration (hjps-ops, hubspot-dataops)

Two pilot repos must run the policy checks in advisory mode with artifacts before enforcement transitions. See the companion task list for day-by-day execution: [PHASE6_TASKS.md](PHASE6_TASKS.md)

Decisions locked in:

- Workspace support: standardize on pnpm 10.x (latest stable) with transitional wrappers for npm. ✅ *hjps-toolkit uses pnpm 10.x*
- Internal dependencies: immediate migration to GitHub Packages (no `file:` deps in CI/PRs). ✅ *Multiple packages published*
- hjps-toolkit: incremental TypeScript migration per package with Changesets; publish after tests/coverage. ✅ *COMPLETE - All packages migrated*

References:

- Phase 2 completion baseline: [PHASE2_COMPLETION.md](../PHASE2_COMPLETION.md)
- Phase 2.5 infrastructure hardening: [PHASE2_5_HANDOFF.md](../PHASE2_5_HANDOFF.md)
- Phase 5 enforcement/exception/metrics status: [PHASE5_COMPLETION_VERIFICATION.md](../PHASE5_COMPLETION_VERIFICATION.md)
- Current iteration execution plan: [EXECUTION_PLAN.md](../EXECUTION_PLAN.md)
- Performance standards (budgets, SLOs): [PERF_STANDARDS.md](../../standards/PERF_STANDARDS.md)
- Gaps analysis (perf, cost, obs not enforced): [analysis/GAPS.md](../analysis/GAPS.md)
- Phase 6 priorities alignment: [analysis/recommendations/phase-6-priorities.md](../analysis/recommendations/phase-6-priorities.md)

---

## Phase Baseline

- Phases 0–2 delivered core skills, templates, and MCP enhancements (green): [PHASE2_COMPLETION.md](../PHASE2_COMPLETION.md)
- Phase 2.5 hardened infra (script runner, config-driven gates): [PHASE2_5_HANDOFF.md](../PHASE2_5_HANDOFF.md)
- Phase 5 added enforcement/exception/metrics and reusable workflow; pilots unverified: [PHASE5_COMPLETION_VERIFICATION.md](../PHASE5_COMPLETION_VERIFICATION.md)
- Execution plan still template-focused; adoption blockers remain: [EXECUTION_PLAN.md](../EXECUTION_PLAN.md)

---

## Workstreams (with Acceptance Criteria)

1) Performance Enforcement (FE + API)

- Deliver skills via CLI: `perf:bundle-budget-check`, `perf:lighthouse-check` (full), `perf:api-latency`.
- Reuse adapters: `scripts/enforce-bundle-budget.ts` (frontend), API latency via traces/fixtures.
- Acceptance:
  - Templates run perf skills locally and in CI; FULL gate auto-fails on budget breaches.
  - hjps-frontend publishes bundle-diff artifacts for ≥3 consecutive PRs.
  - API latency regressions above budgets fail FULL gate (no cold-starts), budgets from [PERF_STANDARDS.md](../../standards/PERF_STANDARDS.md).

2) Workspace Standardization (pnpm 10.x) + File: Dep Migration (GH Packages)

- Deliver skills via CLI: `repo:workspace-check` (npm/pnpm/yarn detect, enforce pnpm 10.x), `deps:file-protocol-check` (advisory→enforce in FULL), `deps:publish-check` (publisher readiness).
- Immediate GH Packages migration per DEPENDENCY_STRATEGY; add `.npmrc` for scope.
- Acceptance:
  - All active repos declare `packageManager: pnpm@10.x` and pass `repo:workspace-check` in FULL.
  - No `file:` deps in CI across adopters; two consecutive green installs.

3) hjps-toolkit Publisher Readiness + Incremental TS Migration ✅ **COMPLETE**

- ✅ Changesets integrated with automated beta and stable release workflows (`.github/workflows/release.yml`)
- ✅ All 13 packages migrated to TypeScript with ESM (`"type": "module"`)
- ✅ Multiple packages published to GitHub Packages
- ✅ Security hardened (4 Dependabot + 30 CodeQL alerts resolved)
- ✅ New capabilities delivered:
  - toolkit-telemetry@0.0.10: auto-backfill + CI tracking
  - toolkit-logger@0.0.6: TimescaleTransport + structured logging
  - toolkit-pdf@0.0.2-alpha.3: TypeScript conversion
  - toolkit-config@0.0.2: Standalone function exports for CLI integration
- ⏳ Remaining: `repo:package-check` skill implementation (infrastructure ready)
- Acceptance Status:
  - ✅ Packages published to GH Packages with semantic versioning
  - ✅ Changesets-driven releases operational
  - ⏳ Per-package coverage thresholds (infrastructure ready, enforcement pending)

4) Cost & Observability Enforcement (Advisory→Warn) ⏳ **IN PROGRESS**

Implements DevOS [§5: Telemetry & Vector Memory](../DEVOS_ARCHITECTURE_CANVAS.md#5-telemetry--vector-memory-time-based-db--embeddings):

- ✅ **Infrastructure Complete (Phases 1-2):**
  - TimescaleDB + pgvector ([schemas](observability/IMPLEMENTATION_PLAN.md#unified-database-schema))
  - Grafana with provisioned datasources
  - OTEL Collector configured for traces, metrics, logs
  - GPU Embeddings Service (`local-llm`) with BAAI/bge-large-en-v1.5
  - Coding Agent Service (`vllm-chat`) with Qwen/Qwen2.5-32B-Instruct-AWQ
  - toolkit-logger@0.0.6 with TimescaleTransport
  - **End-to-end verified:** 41 logs with 41 embeddings successfully stored
- ✅ **Phase 3 In Progress:**
  - hjps-ops services updated to use new logger transport
  - End-to-end log visibility with vector embeddings verified
- ⏳ **Self-Healing Architecture (Phase 7):**
  - Implements DevOS [§6: Self-Healing](../DEVOS_ARCHITECTURE_CANVAS.md#6-self-healing--failure-handling)
  - [4-step state machine](../DEVOS_ARCHITECTURE_CANVAS.md#62-self-healing-state-machine-detect--retrieve--propose--apply): detect → retrieve → propose → apply
  - [Autonomy Level 0](../DEVOS_ARCHITECTURE_CANVAS.md#7-autonomy-levels) (suggest-only)
  - Guardrails: Default disabled (`enabled: false`), no silent mutations, policy validation, SLO limits
- ⏳ **Remaining (Phase 6):**
  - `cost:budget-check` skill implementation
  - `obs:otel-check` skill implementation (infrastructure ready)
  - hjps-frontend telemetry configuration
  - Initial Grafana dashboards (Builds, Tests, Traces)
  - Publish `scripts/metrics-dashboard.ts` output as CI artifact
- **Next Steps:**
  1. **Implement Build Observability:**
     - Create `scripts/build-reporter.mjs` to populate `build_events` (lifecycle status).
     - **NEW:** Integrate `toolkit-logger` into build/test scripts to ship stdout/stderr to `application_logs`.
  2. **Verify Dashboards:** Validate `ci-test-intelligence.json` and `build-deploy-events.json`.
  3. **Validate vLLM Service:** Execute `test-vllm-chat.mjs`.
  4. **Formalize Agent Interface:** Define `AgentPort` and implement `VLLMAgentAdapter`.
  5. **Prototype Self-Healing:** Create Level 0 worker script.
- Acceptance Status:
  - ⏳ Skills produce actionable findings in advisory mode within 2 weeks
  - ⏳ Violations < 5% before flip to warn
  - ✅ Observability infrastructure operational and verified

5) Frontend Development (Week 3) 🆕

- Establish `frontend-toolkit` as production-ready SDK
  - Design system (components, tokens, utilities)
  - Build tooling configuration (Vite/Next.js, Tailwind/CSS)
  - Form validation patterns
  - Publishing to GitHub Packages
- Build GUI applications in hjps-ops
  - **approval-portal** - Replace CLI approval workflow
    - Consume frontend-toolkit components
    - Migrate approval logic from CLI to web UI
  - **timesheet-portal** - Replace CLI timesheet commands
    - Consume frontend-toolkit components
    - Migrate timesheet logic from CLI to web UI
- Acceptance:
  - frontend-toolkit published to GitHub Packages with `0.1.0+` versions
  - ≥2 GUI applications deployed and functional
  - GUI development exposes business logic gaps → refactoring backlog
  - Browser telemetry configured (feeds to observability platform)

6) Toolkit Extraction (Week 4-5) 🆕

- Extract stable Core modules from ops/dataops to toolkits **after** boundary separation
- Target patterns (examples):
  - Approval state machine → `hjps-toolkit/packages/toolkit-workflow`
  - HubSpot query builders → `hjps-toolkit/packages/toolkit-hubspot` (expand existing)
  - Form validation patterns → `frontend-toolkit/packages/validation`
  - Vector pipeline utilities → `vector-toolkit/packages/utils`
- Process:
  - Week 3-4: Mark extraction candidates during refactoring (comments/TODOs)
  - Week 4-5: Review Core modules, extract stable patterns
- Acceptance:
  - ≥3 patterns extracted with clear boundaries
  - Extracted modules have unit tests (≥80% coverage)
  - Original repos depend on extracted packages via registry
  - Extraction rationale documented in ADRs


5) Multi-language (hubspot-dataops) Enablement

- Deliver skills: `repo:language-check` (drive skill selection), `deps:policy-python` (pyproject.toml + lock parsing), `data:migration-check` (checksum validation of DB migration sets).
- Acceptance:
  - Mixed-language checks run with <10% false positives; Python dependency violations surface in CI ≤ 10 minutes after change; migration drift report attached to PRs.

6) Template Guardrails (FE + API)

- Bake `perf:bundle-budget-check`, `api:openapi-check`, and OTel wiring into templates; FE template publishes bundle analysis artifacts by default.
- Acceptance:
  - New template clones pass perf + OpenAPI skills on first run; existing hjps-frontend PRs show zero “missing artifact” findings post-migration.

7) Pilot Adoption Gate (Advisory Mode)

- Require two pilot repos (hjps-ops, hubspot-dataops) to run `policy-checks.yml` in advisory mode with artifacts/ metrics.
- Use metrics to drive enforcement flips; thresholds as in `policy/enforcement-mode.json`.
- Acceptance:
  - ≥3 advisory runs/week per pilot; metrics artifacts visible; flip advisory→warn after 2 weeks under 5% failure rate.

8) Speed Improvements & Gate Refinements

- Change-aware FAST gate (lint/format on changed files; network/build checks only if inputs changed); cache heavy skills by lockfile hash.
- Move `build:validation` to FULL by default; keep parallelization tight.
- Acceptance:
  - FAST gate ≤ 2 minutes cold, ≤ 30s typical on small changes; FULL gate ≤ 7 minutes; `artifacts/metrics.json` shows top-3 slow skills trending down: artifacts/metrics.json:1

---

## Skill Specifications (inputs, outputs, runtime, artifacts)

- perf:bundle-budget-check
  - Inputs: static bundle stats or Next.js analysis output; optional budgets override.
  - Outputs: JSON findings with route, size gz, diff, limit; artifact `artifacts/perf/bundle.json`.
  - Runtime: ≤ 5s; Enforce in FULL.

- perf:lighthouse-check
  - Inputs: route list; budgets for perf/a11y.
  - Outputs: JSON summary per route; artifact `artifacts/perf/lhci.json`.
  - Runtime: 60–120s; FULL-only.

- perf:api-latency
  - Inputs: trace samples or fixture timings; budgets from PERF_STANDARDS.
  - Outputs: JSON with p95 by endpoint; artifact `artifacts/perf/api-latency.json`.
  - Runtime: ≤ 5s; Enforce in FULL.

- api:openapi-check
  - Inputs: openapi config, codegen script, generated types path.
  - Outputs: JSON with schema validity and diff-clean status; artifact `artifacts/api/openapi.json`.
  - Runtime: ≤ 10s; FULL.

- cost:budget-check
  - Inputs: PR time-to-green metrics, embeddings usage caps.
  - Outputs: JSON violations; artifact `artifacts/cost/budgets.json`.
  - Runtime: ≤ 5s; Advisory→Warn.

- obs:otel-check
  - Inputs: code/config; checks required attributes (trace_id, request_id, user_id, pipeline_id, batch_id, duration_ms) and FE traceparent.
  - Outputs: JSON findings; artifact `artifacts/obs/otel.json`.
  - Runtime: ≤ 3s; Advisory→Warn.

- repo:workspace-check
  - Inputs: package.json, lockfile; detects workspace manager; enforces pnpm 10.x.
  - Outputs: JSON findings; artifact `artifacts/repo/workspace.json`.
  - Runtime: ≤ 2s; Enforce in FULL after grace.

- deps:file-protocol-check
  - Inputs: package.json dependencies; reports `file:` usage.
  - Outputs: JSON findings with path and guidance; artifact `artifacts/deps/file-deps.json`.
  - Runtime: ≤ 2s; Advisory→Enforce in FULL.

- deps:publish-check
  - Inputs: publishConfig, .npmrc, workflow presence.
  - Outputs: JSON readiness; artifact `artifacts/deps/publish.json`.
  - Runtime: ≤ 2s; FULL for publisher repos.

- arch:module-system-check
  - Inputs: per-package module type, TS moduleResolution.
  - Outputs: JSON inconsistencies; artifact `artifacts/arch/modules.json`.
  - Runtime: ≤ 3s; FULL.

- repo:language-check / deps:policy-python / data:migration-check
  - Inputs: pyproject/lock, migration directories; outputs JSON findings; artifacts under `artifacts/python/*` and `artifacts/data/*`.
  - Runtime: ≤ 5s each; Advisory.

---

## Gates & Enforcement Updates

- FAST Gate (≤ 2 min goal)
  - Skills: prettier (changed files), eslint (changed files), typecheck, repo:script-map, deps:policy-node; optional vuln-scan only if lockfile changed or cache hit ≤ 1s.
  - Remove: build:validation (move to FULL) unless change-aware trigger fires.

- FULL Gate (≤ 7 min goal; parallel groups)
  - Add: perf skills, api:openapi-check, SBOM+OPA, file-type, secret scan, build validation, workspace/file-protocol/module-system checks, publish-check.

- Enforcement Transitions
  - Advisory→Warn when failureRate < 10% over 14 days; Warn→Enforce when < 5% over 14 days; auto-rollback per policy/enforcement-mode.json:1.

---

## Policy & Constraints Changes

- Node constraints
  - Expand `policy/node/constraints.json` with framework floors (React 19, Next 15.5.x), banned packages (winston, jest, joi), ESLint 9 flat, Vitest required, and extended `mustBeSingleVersion`.

- Documentation TTL
  - Enhance docs freshness to require TTL headers on README, ARCHITECTURE, CONTRIBUTING, SECURITY.

- Mocking enforcement
  - ESLint rule: allow only `vi.mock('node:*')` boundary mocks; forbid logic-layer mocks.

---

## Pilot Rollout & Metrics

- Pilots: hjps-ops, hubspot-dataops run policy-checks.yml in advisory; upload artifacts and metrics dashboards.
- Metrics: Use artifacts/metrics.json and dashboard for failure trends and exception expirations: scripts/metrics-dashboard.ts:1, artifacts/metrics.json:1
- Flip enforcement per thresholds; publish weekly rollup.

---

## Timeline & Sequencing

**Updated:** 2025-11-24 based on [INTEGRATION_ANALYSIS.md](../analysis/INTEGRATION_ANALYSIS.md)

- Week 1 ⏳ **PENDING + CRITICAL ADDITIONS**
  - **Original:** pnpm standardization (wrappers for npm), repo:workspace-check, deps:file-protocol-check, FAST gate change-aware, vuln-scan caching, move build to FULL, gates update
  - **Critical Additions (~3-4 hours):**
    - Fix DataOps broken entry points (30 min)
    - Add missing OTEL core to frontend (15 min)
    - Add prettier to standards-cli (15 min)
    - Expand dependency constraints (Zod v4, banned packages, framework floors) (1 hour)
    - Remove zombie dependencies (30 min)
  - **Note:** hjps-toolkit already uses pnpm 10.x; skill implementations needed

- Week 2 ✅ **MOSTLY COMPLETE** + **SKILL IMPLEMENTATION SPRINT (~20-25 hours)**
  - ✅ GH Packages publishing for hjps-toolkit (multiple packages published with Changesets)
  - ✅ Changesets bootstrap and automated workflows
  - ✅ TypeScript migration complete (all 13 packages)
  - ✅ Security fixes (4 Dependabot + 30 CodeQL alerts)
  - ✅ Observability infrastructure (Phases 1-2 complete)
  - **Critical Skill Implementations:**
    - arch:layer-check (4-6h) - CRITICAL, blocks refactoring validation
    - perf:bundle-budget-check (3-4h) - HIGH, blocks frontend adoption
    - api:openapi-check (4-6h) - HIGH, blocks API adoption
    - repo:package-check (4-6h) - MEDIUM, completes infrastructure
    - Create version manifest (2-3h) - Resolves version sprawl
    - Deploy dep-cruiser configs (2h) - Architecture enforcement
  - ⏳ Templates add perf/OpenAPI guardrails

- Week 3 🚧 **IN PROGRESS** + **PERF/OBS COMPLETION (~15-20 hours)**
  - **Original:**
    - ⏳ arch:module-system-check (ESM standard already implemented across toolkit)
    - ⏳ migrate consumers from file: deps
    - ✅ Observability Phase 3 started (hjps-ops services updated)
  - **Performance & Observability Skills:**
    - perf:lighthouse-check (4-6h) - MEDIUM, FULL-only
    - perf:api-latency (4-6h) - HIGH, hjps-ops adoption
    - obs:otel-check (3-4h) - MEDIUM, infrastructure ready
    - Metrics dashboard artifact (2-3h) - MEDIUM, pilot tracking
  - **Quick Fixes:**
    - Fix duplicate Vitest configs (30 min)
    - Frontend migration to toolkit packages (2-3 days)

- Week 4 ⏳ **PENDING + EXPLICIT TASK ADDITIONS**
  - **Original:**
    - repo:language-check, deps:policy-python, data:migration-check (advisory) for hubspot-dataops
    - ✅ hjps-toolkit TS migration complete (no further per-package work needed)
  - **Explicit Mock & Quality Tasks:**
    - hjps-ops mock reduction: 291 → <20 (1-2 weeks) - CRITICAL
    - dataops mock reduction: 31 → <10 (1 week) - HIGH
    - Python test coverage: 0% → 80% (1-2 weeks) - CRITICAL
    - Standardize TS execution (tsx) (2-3h)
    - Document multi-language boundaries (2-3h)
    - pdf-generator ESM migration (1-2h)

- **Parallel Track (Weeks 3-8):**
  - hjps-toolkit test coverage: 0% → 80% (10-15 days)
  - Assign separate owner to avoid blocking main timeline
  - Critical for quality validation and consumer confidence

---

## Acceptance Criteria (Phase 6)

- Speed ⏳
  - ⏳ FAST gate ≤ 2 min cold, ≤ 30s typical; FULL gate ≤ 7 min; metrics artifacts report top-3 slow skills.
  - **Status:** Performance improvements pending implementation

- Performance ⏳
  - ⏳ Perf skills integrated; FE bundles and Lighthouse budgets enforced; API SLO checks active.
  - **Status:** Skill implementations pending

- Consistency 🚧 **PARTIAL**
  - ✅ pnpm 10.x in hjps-toolkit (workspace configuration complete - latest stable)
  - ✅ GH Packages used for internal deps (multiple packages published)
  - ⏳ Framework floors enforcement pending (React 19, Next 15.x, ESLint 9, Vitest - all latest stable)
  - ⏳ Mocking rule pending
  - ⏳ Doc TTLs validation pending

- Coverage of Gaps 🚧 **PARTIAL**
  - ⏳ Perf skills pending implementation
  - ⏳ Cost skills pending implementation
  - ✅ Observability infrastructure complete (Phases 1-2), skill implementation pending
  - ⏳ Portability/migrations skills pending
  - ⏳ Dependency skills pending
  - ⏳ Governance skills pending

- Adoption ⏳
  - ⏳ Pilots produce ≥3 advisory runs/week with artifacts
  - ⏳ Enforcement flips from advisory→warn after 2 weeks under 5% failure rate
  - **Status:** Pilot integration pending

**Major Accomplishments:**
- ✅ hjps-toolkit: Complete TypeScript migration with ESM, Changesets, and GH Packages publishing
- ✅ Observability: Infrastructure and vector integration complete (41 logs with embeddings verified)
- ✅ Security: 4 Dependabot + 30 CodeQL alerts resolved

---

## Proposed Changes (File Map)

- Gates & policy
  - `policy/gates.json`
  - `policy/node/constraints.json`
  - `policy/test-coverage.json`

- Runner & orchestration
  - `scripts/policy-verify.ts`
  - `scripts/utils/gate-resolver.ts`

- New skills (migrated to `packages/standards-cli`)
  - `src/core/skills/perf-bundle-budget.ts`
  - `src/core/skills/perf-lighthouse-check.ts`
  - `src/core/skills/perf-api-latency.ts`
  - `src/core/skills/api-openapi-check.ts`
  - `src/core/skills/obs-otel-check.ts`
  - `src/core/skills/repo-workspace-check.ts`
  - `src/core/skills/deps-file-protocol-check.ts`
  - `src/core/skills/deps-publish-check.ts`
  - `src/core/skills/arch-module-system-check.ts`
  - `src/core/skills/repo-language-check.ts`
  - `src/core/skills/deps-policy-python.ts`
  - `src/core/skills/data-migration-check.ts`

- Skill updates
  - `scripts/skills/security-vulnerability-scan.ts`
  - `scripts/skills/style-prettier-check.ts`
  - `scripts/skills/style-eslint-check.ts`
  - `scripts/skills/test-coverage.ts`
  - `scripts/skills/deps-policy-node.ts`

- Docs & templates
  - `docs/AGENTIC_IMPLEMENTATION_PLAN.md`
  - `docs/planning/EXECUTION_PLAN.md`
  - `docs/POLICY_REFERENCE.md`
  - `docs/DEPENDENCY_STRATEGY.md`
  - Template READMEs for FE/API (perf/OpenAPI guardrails)

---

## Risks & Mitigations

- Skill proliferation increases complexity
  - Standardize skill inputs/outputs, document runtime budgets, tag fast/full suitability.

- False positives in advisory checks
  - Start advisory; tune with exceptions; escalate based on metrics thresholds.

- hjps-toolkit migration pace
  - JS accommodation if needed during TS migration; publish early; schedule per-package conversion.

---

## Cross-References

**Architecture & Vision:**
- **DevOS Architecture Canvas:** docs/DEVOS_ARCHITECTURE_CANVAS.md
- **DevOS Vision:** docs/AI_NATIVE_DEVOS_VISION.md

**Planning & Implementation:**
- Agentic implementation: docs/AGENTIC_IMPLEMENTATION_PLAN.md
- Observability platform: docs/planning/observability/IMPLEMENTATION_PLAN.md
- Execution plan: docs/planning/EXECUTION_PLAN.md

**Standards & Policy:**
- Observability/cost standards: docs/OBSERVABILITY.md, docs/COST_GUARDS.md
- Perf standards: docs/PERF_STANDARDS.md
- Gaps summary: docs/planning/analysis/GAPS.md
