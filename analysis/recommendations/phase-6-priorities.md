# Phase 6 Implementation Priorities

**Last Updated:** 2025-11-11  
**Policy-Version:** 1.0.0  
**TTL:** 90d  
**Owner:** Architecture + Security

---

## Overview

This document prioritizes Phase 6 implementation tasks based on repository adoption requirements and standards gaps.

---

## Effort Estimation Methodology

**Estimation Basis:**

Effort estimates in this document are based on:
1. **Historical Velocity** - Phases 0-5 implementation experience (1 skill ≈ 1 day avg.)
2. **Complexity Factors** - Lines of code, integration points, test scenarios
3. **Integration Overhead** - +50% for cross-cutting concerns
4. **Single Engineer Assumption** - Focused work, no context switching

**Estimation Formula:**

| Complexity | Base Effort | Integration | Total | Example |
|------------|-------------|-------------|-------|---------|
| **Simple** | 1 day (8 hours) | +0-25% | 1-1.5 days | Pattern matching skill (file: protocol check) |
| **Medium** | 2-3 days (16-24 hours) | +25-50% | 2-4 days | Integration skill (workspace detection + validation) |
| **Complex** | 4-6 days (32-48 hours) | +50-100% | 4-8 days | Orchestration skill (monorepo per-package validation) |

**Complexity Indicators:**

- **Simple:** Single skill, clear pattern, no dependencies, <300 LOC
- **Medium:** Multiple skills, moderate complexity, some integration, 300-800 LOC
- **Complex:** Multiple skills, high complexity, significant integration, >800 LOC

**Skill Implementation Pattern (Standard Structure):**
```
mcp/skills/[namespace]-[name].yml        (~50-100 lines)
scripts/skills/[name].ts                 (~200-300 lines for simple, ~500+ for complex)
scripts/skills/__tests__/[name].test.ts  (~150-250 lines)
```

**Actual Examples from Phases 0-5:**
- `style:jsdoc-coverage` - Simple pattern matching ≈ 1 day
- `security:vulnerability-scan` - npm audit wrapper ≈ 1.5 days
- `test:coverage` - Vitest integration ≈ 2 days
- `build:validation` - TypeScript + package.json checks ≈ 2.5 days

**Assumptions (Not Always Documented):**

1. ✅ **Single engineer working full-time** - Estimates don't account for team size
2. ✅ **No context switching** - Assumes focused work on one task
3. ✅ **Standards established** - Leverages existing patterns from toolkit-standards
4. ✅ **Tooling in place** - TypeScript, Vitest, ESLint, Prettier all configured
5. ✅ **No major blockers** - Assumes infrastructure (CI, repos) is available
6. ⚠️ **Best case estimates** - No buffer for unknowns included

**Confidence Levels:**

| Priority | Confidence | Rationale |
|----------|------------|-----------|
| HIGH Priority | **MEDIUM** (±25%) | Similar to Phase 5 work, patterns established |
| MEDIUM Priority | **MEDIUM-LOW** (±50%) | Some unknowns (multi-language, frontend-specific) |
| LOW Priority | **LOW** (±100%) | Exploratory work, no clear precedent |

**Risk Buffer Recommendations:**

- **HIGH Priority tasks:** Add +25% buffer (e.g., 3-5 days → 4-6 days total)
- **MEDIUM Priority tasks:** Add +50% buffer (e.g., 4-6 days → 6-9 days total)
- **LOW Priority tasks:** Add +100% buffer or defer to Phase 7+ for better scoping

**Validation Plan:**

During Phase 6, track:
- ✅ Actual hours spent per skill
- ✅ Compare estimates vs. actuals
- ✅ Identify estimation patterns (too optimistic? too conservative?)
- ✅ Refine estimation model for Phase 7+
- ✅ Document blockers and unknowns encountered

**Example Estimate Breakdown:**

**Task:** Workspace Support (repo:workspace-check)
- Base implementation: 2 days (npm workspace detection logic)
- Integration with deps:policy-node: 1 day
- Testing & edge cases: 1 day
- Documentation: 0.5 day
- **Total: 4.5 days** (rounded to 3-5 days with confidence range)

---

## 🔴 High Priority (Blocking Adoption)

### 1. Workspace Support (hjps-ops, hjps-toolkit)

**Repositories Affected:** hjps-ops, hjps-toolkit

**Skills Needed:**
- `repo:workspace-check` - Detect npm/pnpm/yarn workspaces
- `deps:policy-node` enhancement - Support npm workspace dependencies
- Lockfile integrity checks for npm workspaces

**Impact:** Blocks hjps-ops and hjps-toolkit adoption

**Estimated Effort:** 3-5 days

---

### 2. Reusable Policy-Checks Workflow

**Repositories Affected:** All adopting repositories

**Requirements:**
- Follow existing reusable workflow pattern
- Support npm and pnpm workspaces
- Handle file: protocol dependencies in CI
- Configurable gate selection (fast/full)

**Impact:** Enables org-wide adoption

**Estimated Effort:** 2-3 days

---

### 3. File: Protocol Dependency Support

**Repositories Affected:** hjps-ops

**Skills Needed:**
- `deps:file-protocol-check` - Validate file: dependencies
- Allow file: dependencies for local development
- Warn about file: dependencies in production builds

**Impact:** Blocks hjps-ops adoption

**Estimated Effort:** 1-2 days

---

### 4. Resolve Testing Policy Conflict

**Repositories Affected:** All repositories

**Requirements:**
- Clarify no-mocks policy scope
- Update implementation plan Section 7 and 10
- Update ESLint rule or policy documentation

**Impact:** Resolves documented conflict

**Estimated Effort:** 1 day

---

## ⚠️ Medium Priority (Enables Full Adoption)

### 5. Monorepo Support (hjps-toolkit)

**Repositories Affected:** hjps-toolkit

**Skills Needed:**
- `repo:monorepo-check` - Lerna/Changesets validation
- `repo:package-check` - Per-package validation
- `deps:internal-check` - Internal package dependencies

**Impact:** Enables hjps-toolkit full adoption

**Estimated Effort:** 4-6 days

---

### 6. Mixed Module System Support (hjps-ops)

**Repositories Affected:** hjps-ops

**Skills Needed:**
- `arch:module-system-check` - CommonJS/ESM validation
- Support mixed module systems in policy checks
- Verify typecheck works with NodeNext resolution

**Impact:** Enables hjps-ops full adoption

**Estimated Effort:** 2-3 days

---

### 7. Multi-Language Support (hubspot-dataops)

**Repositories Affected:** hubspot-dataops

**Skills Needed:**
- `repo:language-check` - Python/Node hybrid detection
- Enhance `deps:policy-python` for comprehensive validation
- Support Python package management validation

**Impact:** Enables hubspot-dataops adoption

**Estimated Effort:** 3-4 days

---

### 8. Frontend-Specific Skills (hjps-frontend)

**Repositories Affected:** hjps-frontend

**Skills Needed:**
- `perf:bundle-budget-check` - Bundle size validation (25KB per route)
- `api:openapi-check` - OpenAPI client generation validation
- `perf:lighthouse-check` - Web Vitals validation

**Impact:** Enables hjps-frontend full adoption

**Estimated Effort:** 4-5 days

---

### 9. Performance Skills (General)

**Repositories Affected:** All frontend/API repositories

**Skills Needed:**
- `perf:budget-check` - Bundle size validation (general)
- `perf:api-latency` - API SLO validation

**Impact:** Enforces performance standards

**Estimated Effort:** 2-3 days

---

### 10. Vector-Toolkit Specific Skills

**Repositories Affected:** vector-toolkit

**Skills Needed:**
- `repo:workspace-check` - pnpm workspace detection (nested packages)
- `repo:package-check` - Per-package validation
- `cost:embedding-budget` - Embedding cost validation (optional)

**Impact:** Enables vector-toolkit full adoption

**Estimated Effort:** 3-4 days

---

### 11. Dependency Policy Enhancements

**Repositories Affected:** All repositories

**Skills Needed:**
- Framework version validation (React 19, Next 15.5.4, etc.)
- Tooling validation (ESLint v9, Vitest)
- Update `policy/node/constraints.json` with version floors

**Impact:** Enforces dependency standards

**Estimated Effort:** 2-3 days

---

## 📋 Low Priority (Future Enhancements)

### 10. Cost Guard Skills
- `cost:budget-check` - Cost tripwire validation
- `cost:embedding-budget` - Embeddings monitoring

### 11. Observability Skills
- `obs:otel-check` - OTel instrumentation validation
- `obs:attribute-check` - Required attributes validation

### 12. Portability Skills
- `portability:drill-check` - Quarterly drill validation
- `portability:evidence-check` - Evidence storage validation

### 13. Security Skills
- `security:auth-check` - OIDC/PKCE validation
- `security:webhook-check` - Signature verification
- `security:rbac-check` - RBAC claims validation

### 14. Governance Skills (Optional)
- `repo:branch-naming` - Branch name validation
- `repo:pr-checklist` - PR requirements validation

### 15. Database/Data Layer Skills (Optional)
- `data:postgres-check` - PostgreSQL schema standards
- `data:schema-registry-check` - Schema registry compliance
- `data:export-check` - Data export/import tooling

---

## Implementation Timeline with Critical Path

### Critical Path Analysis

```
CRITICAL PATH (Week 1-6):
Testing Policy → Workspace Support → Reusable Workflow → Repository Adoptions

Week 1:
┌─────────────────────────────────────────┐
│ [COMPLETED] Testing Policy Resolution  │ ← BLOCKS ALL (credibility issue)
│ Effort: 1 day                          │
└─────────────────────────────────────────┘
        ↓
┌─────────────────────────────────────────┐
│ Workspace Support Implementation       │ ← BLOCKS hjps-ops, hjps-toolkit, vector-toolkit
│ - repo:workspace-check (1-2 days)      │
│ Effort: 3-5 days                       │
└─────────────────────────────────────────┘

Week 2 (Parallel with Week 1 Workspace):
┌──────────────────────────────────┐     ┌──────────────────────────────────┐
│ File: Protocol Support           │     │ Reusable Policy-Checks Workflow  │
│ - deps:file-protocol-check       │     │ - GitHub Actions workflow        │
│ Effort: 1-2 days                 │     │ Effort: 2-3 days                 │
└──────────────────────────────────┘     └──────────────────────────────────┘
        ↓ (BLOCKS hjps-ops, hubspot-dataops)     ↓ (BLOCKS all adoptions)

Week 3-4:
┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐
│ Monorepo Support     │  │ Mixed Module System  │  │ Multi-Language       │
│ - repo:monorepo      │  │ - arch:module-system │  │ - repo:language      │
│ Effort: 4-6 days     │  │ Effort: 2-3 days     │  │ Effort: 3-4 days     │
│ BLOCKS: hjps-toolkit │  │ BLOCKS: hjps-ops     │  │ BLOCKS: hubspot-data │
└──────────────────────┘  └──────────────────────┘  └──────────────────────┘

Week 5-6 (Can run in parallel):
┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐
│ Frontend Skills      │  │ Performance Skills   │  │ Dependency Enhance   │
│ - bundle-budget      │  │ - lighthouse-check   │  │ - framework-check    │
│ - openapi-check      │  │ - api-latency        │  │ - tooling-check      │
│ Effort: 4-5 days     │  │ Effort: 2-3 days     │  │ Effort: 2-3 days     │
│ BLOCKS: hjps-frontend│  │ BLOCKS: none         │  │ BLOCKS: none         │
└──────────────────────┘  └──────────────────────┘  └──────────────────────┘
```

### Detailed Week-by-Week Timeline

#### Week 1: Foundation & Critical Path Start

**Day 1: ✅ COMPLETED (2025-11-11)**
- [x] Testing Policy Conflict Resolution
- [x] Update AGENTIC_IMPLEMENTATION_PLAN.md
- [x] Update documentation references
- **Status:** ✅ COMPLETE

**Days 2-5: Workspace Support (CRITICAL PATH)**
- [ ] Design `repo:workspace-check` skill
- [ ] Implement npm/pnpm/yarn detection
- [ ] Test with hjps-ops (npm), hjps-toolkit (npm+Lerna), vector-toolkit (pnpm)
- [ ] Integration with deps:policy-node
- **Effort:** 3-5 days
- **Blocks:** hjps-ops, hjps-toolkit, vector-toolkit adoption
- **Dependencies:** None
- **Can Run in Parallel With:** File: protocol support, reusable workflow

**Days 3-5: File: Protocol Support (PARALLEL)**
- [ ] Design `deps:file-protocol-check` skill
- [ ] Implement file: protocol detection
- [ ] Add warnings for CI usage
- [ ] Test with hjps-ops and hubspot-dataops
- **Effort:** 1-2 days
- **Blocks:** hjps-ops, hubspot-dataops adoption
- **Dependencies:** None
- **Can Run in Parallel With:** Workspace support

#### Week 2: Infrastructure Enablement

**Days 1-3: Reusable Policy-Checks Workflow (CRITICAL PATH)**
- [ ] Design reusable GitHub Actions workflow
- [ ] Support npm and pnpm workspaces
- [ ] Handle file: protocol dependencies in CI
- [ ] Test with toolkit-standards
- **Effort:** 2-3 days
- **Blocks:** ALL repository adoptions
- **Dependencies:** Workspace support (partial - can start in parallel)

**Days 4-5: Dependency Strategy Implementation**
- [ ] Set up GitHub Packages publishing for hjps-toolkit
- [ ] Configure automated publishing workflow
- [ ] Publish initial versions
- [ ] Document authentication setup
- **Effort:** 1-2 days
- **Blocks:** hjps-ops, hubspot-dataops (file: protocol migration)
- **Dependencies:** None
- **Can Run in Parallel With:** Reusable workflow

**End of Week 2 Checkpoint:**
- ✅ Workspace support implemented
- ✅ File: protocol support implemented
- ✅ Reusable workflow created
- ✅ hjps-toolkit packages published to registry
- **Status:** Ready to begin repository-specific skills

#### Week 3: Repository-Specific Skills (Parallel Execution)

**Track A: hjps-toolkit (Days 1-6)**
- [ ] Day 1-4: Implement `repo:monorepo-check` (Lerna/Changesets validation)
- [ ] Day 5-6: Implement `deps:internal-check` (internal package dependencies)
- **Effort:** 4-6 days total
- **Blocks:** hjps-toolkit adoption
- **Dependencies:** Workspace support ✅
- **Can Run in Parallel With:** Track B, Track C

**Track B: hjps-ops (Days 1-3)**
- [ ] Day 1-3: Implement `arch:module-system-check` (CJS/ESM validation)
- **Effort:** 2-3 days
- **Blocks:** hjps-ops adoption
- **Dependencies:** Workspace support ✅, File: protocol support ✅
- **Can Run in Parallel With:** Track A, Track C

**Track C: hubspot-dataops (Days 1-4)**
- [ ] Day 1-2: Implement `repo:language-check` (multi-language detection)
- [ ] Day 3-4: Enhance `deps:policy-python` (pyproject.toml support)
- **Effort:** 3-4 days total
- **Blocks:** hubspot-dataops adoption
- **Dependencies:** File: protocol support ✅
- **Can Run in Parallel With:** Track A, Track B

**End of Week 3 Checkpoint:**
- ✅ Monorepo support ready (hjps-toolkit can adopt)
- ✅ Mixed module system support ready (hjps-ops can adopt)
- ✅ Multi-language support ready (hubspot-dataops can adopt)

#### Week 4: Frontend & Performance Skills (Parallel Execution)

**Track A: Frontend Skills (Days 1-5)**
- [ ] Day 1-3: Implement `perf:bundle-budget-check`
- [ ] Day 4-5: Implement `api:openapi-check`
- **Effort:** 4-5 days total
- **Blocks:** hjps-frontend adoption
- **Dependencies:** None
- **Can Run in Parallel With:** Track B, Track C

**Track B: Performance Skills (Days 1-3)**
- [ ] Day 1-2: Implement `perf:lighthouse-check`
- [ ] Day 3: Implement `perf:api-latency`
- **Effort:** 2-3 days total
- **Blocks:** None (advisory)
- **Dependencies:** None
- **Can Run in Parallel With:** Track A, Track C

**Track C: Dependency Enhancements (Days 1-3)**
- [ ] Day 1-2: Implement `deps:framework-check`
- [ ] Day 3: Update policy/node/constraints.json
- **Effort:** 2-3 days total
- **Blocks:** None (advisory)
- **Dependencies:** deps:policy-node
- **Can Run in Parallel With:** Track A, Track B

**End of Week 4 Checkpoint:**
- ✅ All HIGH priority skills implemented
- ✅ Frontend skills ready (hjps-frontend can adopt)
- ✅ Performance validation ready

#### Week 5-6: Repository Adoptions & Validation

**Week 5: Pilot Repository Adoptions**
- [ ] hjps-frontend adoption (lowest risk, Days 1-2)
  - Add reusable workflow
  - Configure policy
  - Run in advisory mode
  - Monitor metrics

- [ ] vector-toolkit adoption (after Node version fix, Days 3-4)
  - Fix Node version requirement
  - Add reusable workflow
  - Run in advisory mode

- [ ] hjps-toolkit adoption (after monorepo skills, Days 3-5)
  - Add reusable workflow
  - Per-package validation
  - Run in advisory mode

**Week 6: Core Infrastructure & High-Risk Adoptions**
- [ ] hjps-ops adoption (after all dependencies, Days 1-3)
  - Workspace validation
  - Mixed module validation
  - File: protocol handled
  - Run in advisory mode

- [ ] hubspot-dataops adoption (multi-language, Days 4-6)
  - Python validation
  - Node validation
  - File: protocol handled
  - Run in advisory mode (extended)

- [ ] Metrics collection and analysis (ongoing)

**End of Week 6 Checkpoint:**
- ✅ All repositories have policy system installed
- ✅ All running in advisory mode
- ✅ Metrics being collected
- **Status:** Ready to transition to warn mode (Week 7+)

---

### Critical Path Summary

**Critical Path Skills (Must complete sequentially):**
1. **Week 1:** Testing Policy ✅ → Workspace Support → File: Protocol Support
2. **Week 2:** Reusable Workflow (depends on Workspace Support partial completion)
3. **Week 3:** Repository-specific skills (parallel, depend on Week 1-2 completion)
4. **Week 4:** Frontend & Performance skills (parallel, independent)
5. **Week 5-6:** Repository adoptions (sequential, based on dependencies)

**Total Critical Path Duration:** 6 weeks (from start to all repos in advisory mode)

**Longest Dependency Chain:**
```
Testing Policy (Day 1)
  → Workspace Support (Days 2-5)
    → Reusable Workflow (Days 6-8)
      → Monorepo Support (Days 11-16)
        → hjps-toolkit Adoption (Days 26-29)

Timeline: ~29 days (6 weeks)
```

**Parallel Execution Opportunities:**
- Week 1: Workspace Support + File: Protocol Support (save 1-2 days)
- Week 3: All repository-specific skills (save 6-8 days)
- Week 4: All frontend/performance skills (save 4-6 days)
- Week 5-6: hjps-frontend + vector-toolkit + hjps-toolkit (save 3-4 days)

**Optimized Timeline with Parallelization:** 4-5 weeks (vs. 6 weeks sequential)

---

### Dependency Matrix

| Skill | Depends On | Blocks | Can Run in Parallel With |
|-------|-----------|--------|--------------------------|
| `repo:workspace-check` | None | hjps-ops, hjps-toolkit, vector-toolkit | File: protocol, Workflow |
| `deps:file-protocol-check` | None | hjps-ops, hubspot-dataops | Workspace, Workflow |
| Reusable Workflow | Workspace (partial) | ALL adoptions | File: protocol |
| `repo:monorepo-check` | Workspace | hjps-toolkit | Module-system, Language, Frontend |
| `arch:module-system-check` | Workspace | hjps-ops | Monorepo, Language, Frontend |
| `repo:language-check` | File: protocol | hubspot-dataops | Monorepo, Module-system, Frontend |
| `perf:bundle-budget-check` | None | hjps-frontend | All other frontend/perf |
| `api:openapi-check` | None | hjps-frontend | All other frontend/perf |

**Critical Path (Longest Chain):**
Testing Policy → Workspace Support → Reusable Workflow → Monorepo Support → hjps-toolkit Adoption

**Duration:** 1 + 5 + 3 + 6 + 3 = 18 days (sequential)
**Optimized:** 1 + 5 + 3 + 6 = 15 days (partial parallelization)

**Optimization Recommendation:**
- Start Workspace Support and File: Protocol Support on same day (Day 2)
- Start Reusable Workflow on Day 4 (doesn't need Workspace fully complete)
- Execute all Week 3 skills in parallel (3 engineers or sequential 4-6 days)
- Execute all Week 4 skills in parallel (3 engineers or sequential 2-3 days)

---

### Resource Allocation Scenarios

#### Scenario 1: Single Engineer (Sequential Execution)

**Timeline:** 8-10 weeks
- Week 1: Testing policy + Workspace support
- Week 2: File: protocol + Reusable workflow
- Week 3-4: Monorepo + Module-system + Language support (sequential)
- Week 5: Frontend skills
- Week 6: Performance + Dependency enhancements
- Week 7-10: Repository adoptions + monitoring

**Pros:** Lower coordination overhead, clearer focus
**Cons:** Longer timeline, no parallelization benefits

#### Scenario 2: 2 Engineers (Parallel on Independent Tracks)

**Timeline:** 5-6 weeks
- Week 1: Eng 1: Testing + Workspace | Eng 2: File: protocol + Workflow
- Week 2-3: Eng 1: Monorepo + Module-system | Eng 2: Language support + Frontend
- Week 4: Eng 1: Performance | Eng 2: Dependency enhancements
- Week 5-6: Both: Repository adoptions

**Pros:** Faster timeline, maintains focus, manageable coordination
**Cons:** Requires 2 engineers, moderate coordination needs

#### Scenario 3: 3 Engineers (Maximum Parallelization)

**Timeline:** 4-5 weeks
- Week 1: Split foundation skills across 3 engineers
- Week 2-3: Each engineer owns 1 repository track (toolkit/ops/dataops)
- Week 4: Each engineer owns frontend/perf/deps tracks
- Week 5: All: Repository adoptions

**Pros:** Fastest timeline, maximum parallelization
**Cons:** High coordination overhead, potential merge conflicts

**Recommended Scenario:** **Scenario 2** (2 Engineers, 5-6 weeks)
- Balances speed with coordination overhead
- Enables critical path optimization
- Reduces bottlenecks

---

### Phase 6 Sprint 1 (Weeks 1-2): Foundation

**Week 1 Goals:**
- ✅ **COMPLETED:** Testing Policy Conflict Resolution (Day 1)
- [ ] **CRITICAL:** Workspace Support Implementation (Days 2-5)
  - `repo:workspace-check` skill
  - Integration with deps:policy-node
  - Testing with npm/pnpm/yarn workspaces
- [ ] **PARALLEL:** File: Protocol Support (Days 3-4)
  - `deps:file-protocol-check` skill
  - Warning/advisory mode for CI

**Week 2 Goals:**
- [ ] **CRITICAL:** Reusable Policy-Checks Workflow (Days 1-3)
  - GitHub Actions reusable workflow
  - Support for npm/pnpm workspaces
  - Gate selection (fast/full)
- [ ] **PARALLEL:** Dependency Strategy Implementation (Days 4-5)
  - Set up GitHub Packages publishing
  - Publish hjps-toolkit packages
  - Document authentication setup

**Week 1-2 Deliverables:**
- ✅ Testing policy documented and resolved
- 🔄 Workspace support ready
- 🔄 File: protocol support ready
- 🔄 Reusable workflow available
- 🔄 hjps-toolkit packages published to registry

**Dependencies for Week 3:** Workspace support and reusable workflow must be complete

---

### Phase 6 Sprint 2 (Weeks 3-4): Repository-Specific Skills

**Week 3 Goals (Parallel Execution):**

**Track A: hjps-toolkit Skills (Days 1-6)**
- [ ] Days 1-4: Implement `repo:monorepo-check`
  - Lerna/Changesets validation
  - Independent versioning check
  - Publishing configuration validation
- [ ] Days 5-6: Implement `deps:internal-check`
  - Internal package dependency validation
  - Version consistency checks

**Track B: hjps-ops Skills (Days 1-3)**
- [ ] Days 1-3: Implement `arch:module-system-check`
  - CommonJS/ESM detection
  - NodeNext module resolution validation
  - Mixed module warnings/documentation

**Track C: hubspot-dataops Skills (Days 1-4)**
- [ ] Days 1-2: Implement `repo:language-check`
  - Python/Node detection
  - Multi-language stack validation
- [ ] Days 3-4: Enhance `deps:policy-python`
  - pyproject.toml parsing
  - Python dependency validation
  - pytest configuration validation

**Week 3 Deliverables:**
- 🔄 Monorepo support complete
- 🔄 Mixed module system support complete
- 🔄 Multi-language support complete
- **Status:** hjps-toolkit, hjps-ops, hubspot-dataops READY for adoption

**Week 4 Goals (Parallel Execution):**

**Track A: Frontend Skills (Days 1-5)**
- [ ] Days 1-3: Implement `perf:bundle-budget-check`
  - Next.js bundle analysis integration
  - Budget validation (25KB per route)
  - Warnings and reporting
- [ ] Days 4-5: Implement `api:openapi-check`
  - OpenAPI client generation validation
  - Type safety verification

**Track B: Performance Skills (Days 1-3)**
- [ ] Days 1-2: Implement `perf:lighthouse-check`
  - Lighthouse CI integration
  - Web Vitals validation
  - Performance score thresholds
- [ ] Day 3: Design `perf:api-latency` (defer implementation to Phase 7)

**Track C: Dependency Enhancements (Days 1-3)**
- [ ] Days 1-2: Implement `deps:framework-check`
  - React 19 version validation
  - Next.js 15.5.4 version validation
  - Framework version floors
- [ ] Day 3: Update `policy/node/constraints.json`
  - Add framework version floors
  - Document enforcement strategy

**Week 4 Deliverables:**
- 🔄 Frontend skills complete
- 🔄 Performance validation ready
- 🔄 Dependency policy enhanced
- **Status:** hjps-frontend READY for adoption

---

### Phase 6 Sprint 3 (Weeks 5-6): Repository Adoptions

**Week 5: Low-Risk Adoptions**

**hjps-frontend Adoption (Days 1-2):**
- [ ] Day 1: Add reusable workflow, configure policy, test locally
- [ ] Day 2: Create PR, run in advisory mode, review metrics
- **Risk:** LOW-MEDIUM
- **Owner:** Frontend team

**vector-toolkit Adoption (Days 3-4):**
- [ ] Day 3: Fix Node version (>=24.0.0), add reusable workflow
- [ ] Day 4: Configure policy, test locally, create PR
- **Risk:** LOW-MEDIUM
- **Owner:** Data team
- **Prerequisite:** Node version fix (~1 hour)

**hjps-toolkit Adoption (Days 3-5):**
- [ ] Day 3: Add reusable workflow, configure per-package validation
- [ ] Day 4: Test monorepo validation, internal dependencies
- [ ] Day 5: Create PR, run in advisory mode
- **Risk:** MEDIUM
- **Owner:** Toolkit team

**Week 5 Monitoring:**
- Daily metrics review for all adopted repositories
- Identify and document exceptions
- Monitor FAST gate execution time
- Track failure rates

**Week 6: High-Risk Adoptions**

**hjps-ops Adoption (Days 1-3):**
- [ ] Day 1: Migrate file: protocol dependencies to registry
- [ ] Day 2: Add reusable workflow, configure policy
- [ ] Day 3: Test workspace validation, mixed modules, create PR
- **Risk:** MEDIUM-HIGH
- **Owner:** Ops team
- **Prerequisites:** file: protocol migration complete

**hubspot-dataops Adoption (Days 4-6):**
- [ ] Day 4: Migrate file: protocol dependencies to registry
- [ ] Day 5: Add reusable workflow, configure multi-language policy
- [ ] Day 6: Test Python and Node validation separately, create PR
- **Risk:** HIGH
- **Owner:** Data team
- **Prerequisites:** file: protocol migration, multi-language support
- **Special:** Extended advisory mode (4+ weeks recommended)

**Week 6 Monitoring:**
- Metrics collection for all repositories
- Exception documentation
- Failure pattern analysis
- Success rate trends

**End of Week 6 Checkpoint:**
- ✅ All 5 repositories adopted (toolkit-standards + 4 new)
- ✅ All in advisory mode
- ✅ Metrics collected for 1-2 weeks
- **Status:** Ready to begin warn mode transition (Phase 6.5)

---

### Phase 6+ (Weeks 7+): Stabilization & Future Skills

**Weeks 7-8: Advisory Mode Monitoring**
- Daily metrics review
- Exception management
- Documentation updates
- Team training

**Weeks 9-10: Warn Mode Transition**
- Transition repositories to warn mode (one at a time)
- Monitor failure rates
- Address exceptions
- Validate team readiness

**Weeks 11+: Enforce Mode Transition**
- Transition lowest-risk repositories first (hjps-frontend, vector-toolkit)
- Monitor impact on PR velocity
- Gradual rollout to higher-risk repositories
- Full enforcement by end of Phase 6

**Future Skills (Phase 7+):**
- Cost guard skills (cost:budget-check, cost:embedding-budget)
- Observability skills (obs:otel-check, obs:attribute-check)
- Portability skills (portability:drill-check)
- Additional governance skills

---

### Success Criteria by Week

| Week | Success Criteria | Exit Criteria |
|------|-----------------|---------------|
| **Week 1** | Workspace support working, File: protocol detected | Can validate hjps-ops workspace locally |
| **Week 2** | Reusable workflow runs, hjps-toolkit published | Can run policy checks in CI for any repo |
| **Week 3** | All repo-specific skills working | hjps-toolkit, hjps-ops, hubspot-dataops can run locally |
| **Week 4** | Frontend skills working | hjps-frontend can run policy checks |
| **Week 5** | 3+ repos adopted in advisory mode | Metrics collected for low-risk repos |
| **Week 6** | All 5 repos adopted in advisory mode | Metrics collected for all repos |
| **Week 8+** | Begin warn mode transitions | Failure rates <10%, team trained |

---

### Risks to Timeline

**Potential Delays:**
1. **GitHub Packages setup issues** (+1-2 days) - Registry configuration, authentication
2. **npm workspace edge cases** (+1-2 days) - Unexpected workspace patterns
3. **Multi-language validation complexity** (+2-3 days) - Python/Node integration issues
4. **Team availability** (+1-5 days) - Vacations, competing priorities
5. **Unexpected repository patterns** (+1-3 days) - Edge cases not discovered in analysis

**Buffer Recommendations:**
- Add +25% buffer to critical path (6 weeks → 7.5 weeks)
- Add +2 weeks for warn mode transition
- Add +2 weeks for enforce mode transition
- **Total Phase 6 Timeline:** 11-12 weeks (including full enforcement)

---

## Success Metrics

- [ ] hjps-ops can adopt policy system
- [ ] hjps-toolkit can adopt policy system
- [ ] hubspot-dataops can adopt policy system (with Python support)
- [ ] hjps-frontend can adopt policy system (with frontend-specific skills)
- [ ] vector-toolkit can adopt policy system (with embedding-specific skills)
- [ ] All repositories using reusable policy-checks workflow
- [ ] PR failure rate < 5% after adoption
- [ ] Performance standards enforced
- [ ] Dependency standards fully enforced

---

**See Also:**
- Complete Review: `../../IMPLEMENTATION_PLAN_STANDARDS_REVIEW.md`
- Summary: `../SUMMARY.md`
- Repository Analysis: `../repositories/`

