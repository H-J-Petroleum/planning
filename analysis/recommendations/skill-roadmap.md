# Skill Roadmap - New Skills Needed for Adoption

**Last Updated:** 2025-11-11  
**Policy-Version:** 1.0.0  
**TTL:** 90d  
**Owner:** Architecture + Security

---

## Overview

This document catalogs all new skills needed to enable repository adoption and fill standards gaps.

---

## 🔴 High Priority Skills (Phase 6)

### Repository Skills

#### `repo:workspace-check`
- **Purpose:** Detect workspace type (npm/pnpm/yarn)
- **Repositories:** hjps-ops, hjps-toolkit
- **Priority:** HIGH
- **Dependencies:** None
- **Gate:** FAST

#### `repo:monorepo-check`
- **Purpose:** Validate Lerna/Changesets configuration
- **Repositories:** hjps-toolkit
- **Priority:** HIGH
- **Dependencies:** None
- **Gate:** FAST

#### `repo:package-check`
- **Purpose:** Validate each package independently in monorepo
- **Repositories:** hjps-toolkit
- **Priority:** HIGH
- **Dependencies:** None
- **Gate:** FAST

#### `repo:language-check`
- **Purpose:** Detect multi-language repos (Python/Node hybrid)
- **Repositories:** hubspot-dataops
- **Priority:** HIGH
- **Dependencies:** None
- **Gate:** FAST

### Dependency Skills

#### `deps:file-protocol-check`
- **Purpose:** Validate file: protocol dependencies
- **Repositories:** hjps-ops
- **Priority:** HIGH
- **Dependencies:** None
- **Gate:** FAST

#### `deps:internal-check`
- **Purpose:** Validate internal package dependencies in monorepo
- **Repositories:** hjps-toolkit
- **Priority:** HIGH
- **Dependencies:** None
- **Gate:** FAST

#### `deps:framework-check`
- **Purpose:** Validate framework versions (React 19, Next 15.5.4, etc.)
- **Repositories:** All
- **Priority:** MEDIUM
- **Dependencies:** `deps:policy-node`
- **Gate:** FULL

#### `deps:tooling-check`
- **Purpose:** Validate tooling versions (ESLint v9, Vitest)
- **Repositories:** All
- **Priority:** MEDIUM
- **Dependencies:** None
- **Gate:** FULL

### Architecture Skills

#### `arch:module-system-check`
- **Purpose:** Detect and validate module systems (CommonJS/ESM)
- **Repositories:** hjps-ops
- **Priority:** HIGH
- **Dependencies:** None
- **Gate:** FULL

### Frontend Skills

#### `perf:bundle-budget-check`
- **Purpose:** Validate bundle sizes against budgets
- **Repositories:** hjps-frontend
- **Priority:** HIGH
- **Dependencies:** Build artifacts, nextjs-bundle-analysis
- **Gate:** FULL

#### `api:openapi-check`
- **Purpose:** Validate OpenAPI client generation
- **Repositories:** hjps-frontend
- **Priority:** HIGH
- **Dependencies:** openapi-typescript
- **Gate:** FAST

#### `perf:lighthouse-check`
- **Purpose:** Validate Lighthouse scores (Web Vitals)
- **Repositories:** hjps-frontend
- **Priority:** MEDIUM
- **Dependencies:** Lighthouse CLI
- **Gate:** FULL

#### `frontend:nextjs-check`
- **Purpose:** Validate Next.js App Router patterns
- **Repositories:** hjps-frontend
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

---

## ⚠️ Medium Priority Skills (Phase 6)

### Performance Skills

#### `perf:budget-check`
- **Purpose:** Validate bundle sizes against budgets
- **Repositories:** Frontend repos
- **Priority:** MEDIUM
- **Dependencies:** Build artifacts
- **Gate:** FULL

#### `perf:lighthouse-check`
- **Purpose:** Validate Lighthouse scores (Web Vitals)
- **Repositories:** Frontend repos
- **Priority:** MEDIUM
- **Dependencies:** Lighthouse CLI
- **Gate:** FULL

#### `perf:api-latency`
- **Purpose:** Validate API latency SLOs
- **Repositories:** API repos
- **Priority:** MEDIUM
- **Dependencies:** Metrics integration
- **Gate:** FULL

### Security Skills

#### `security:policy-check`
- **Purpose:** Validate SECURITY.md presence and content
- **Repositories:** All
- **Priority:** MEDIUM
- **Dependencies:** None
- **Gate:** FULL

---

## 📋 Low Priority Skills (Phase 7+)

### Cost Guard Skills

#### `cost:budget-check`
- **Purpose:** Validate cost tripwires
- **Repositories:** All
- **Priority:** LOW
- **Dependencies:** Cost metrics
- **Gate:** FULL (advisory)

#### `cost:embedding-budget`
- **Purpose:** Monitor embeddings budget
- **Repositories:** AI/ML repos
- **Priority:** LOW
- **Dependencies:** Embedding metrics
- **Gate:** FULL (advisory)

### Observability Skills

#### `obs:otel-check`
- **Purpose:** Validate OTel instrumentation
- **Repositories:** All
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

#### `obs:attribute-check`
- **Purpose:** Validate required OTel attributes
- **Repositories:** All
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

### Portability Skills

#### `portability:drill-check`
- **Purpose:** Validate quarterly drills completed
- **Repositories:** All
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

#### `portability:evidence-check`
- **Purpose:** Validate evidence storage
- **Repositories:** All
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

### Security Skills

#### `security:auth-check`
- **Purpose:** Validate OIDC/PKCE implementation
- **Repositories:** Auth-enabled repos
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

#### `security:webhook-check`
- **Purpose:** Validate webhook signature verification
- **Repositories:** Webhook-enabled repos
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

#### `security:rbac-check`
- **Purpose:** Validate RBAC claims
- **Repositories:** RBAC-enabled repos
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

### Governance Skills

#### `repo:branch-naming`
- **Purpose:** Validate branch naming conventions
- **Repositories:** All (optional)
- **Priority:** LOW
- **Dependencies:** Git
- **Gate:** FAST (advisory)

#### `repo:pr-checklist`
- **Purpose:** Validate PR requirements
- **Repositories:** All (optional)
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FAST (advisory)

### Data Layer Skills

#### `data:postgres-check`
- **Purpose:** Validate PostgreSQL schema standards
- **Repositories:** hubspot-dataops
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

#### `data:schema-registry-check`
- **Purpose:** Validate schema registry compliance
- **Repositories:** hubspot-dataops
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

#### `data:export-check`
- **Purpose:** Validate data export/import tooling
- **Repositories:** hubspot-dataops
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

### Infrastructure Skills

#### `infra:docker-check`
- **Purpose:** Validate Docker standards
- **Repositories:** Docker-enabled repos
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

#### `infra:service-check`
- **Purpose:** Validate service configurations
- **Repositories:** Service-enabled repos
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

### Integration Skills

#### `repo:integration-check`
- **Purpose:** Validate inter-repo integration patterns
- **Repositories:** All
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

#### `repo:ecosystem-check`
- **Purpose:** Validate ecosystem relationships
- **Repositories:** All
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

### Publishing Skills

#### `deps:publish-check`
- **Purpose:** Validate package publishing configuration
- **Repositories:** Publishing repos
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

### Toolkit Skills

#### `toolkit:standards-check`
- **Purpose:** Validate toolkit standards compliance
- **Repositories:** Toolkit repos
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

### Migration Skills

#### `migration:tryton-readiness-check`
- **Purpose:** Validate Tryton migration readiness
- **Repositories:** hubspot-dataops
- **Priority:** LOW
- **Dependencies:** None
- **Gate:** FULL (advisory)

---

## Skill Implementation Priority Matrix

| Skill | Priority | Repositories | Phase | Effort |
|-------|----------|--------------|-------|--------|
| `repo:workspace-check` | HIGH | hjps-ops, hjps-toolkit | 6 | 1-2 days |
| `repo:monorepo-check` | HIGH | hjps-toolkit | 6 | 2-3 days |
| `repo:package-check` | HIGH | hjps-toolkit | 6 | 2-3 days |
| `repo:language-check` | HIGH | hubspot-dataops | 6 | 1-2 days |
| `deps:file-protocol-check` | HIGH | hjps-ops | 6 | 1-2 days |
| `deps:internal-check` | HIGH | hjps-toolkit | 6 | 2-3 days |
| `arch:module-system-check` | HIGH | hjps-ops | 6 | 2-3 days |
| `perf:bundle-budget-check` | HIGH | hjps-frontend | 6 | 2-3 days |
| `api:openapi-check` | HIGH | hjps-frontend | 6 | 1-2 days |
| `repo:workspace-check` | HIGH | vector-toolkit | 6 | 1-2 days |
| `repo:package-check` | HIGH | vector-toolkit | 6 | 2-3 days |
| `perf:lighthouse-check` | MEDIUM | hjps-frontend | 6 | 2-3 days |
| `perf:budget-check` | MEDIUM | Frontend | 6 | 2-3 days |
| `security:policy-check` | MEDIUM | All | 6 | 1-2 days |
| `deps:framework-check` | MEDIUM | All | 6 | 2-3 days |
| `cost:budget-check` | LOW | All | 7+ | 2-3 days |
| `obs:otel-check` | LOW | All | 7+ | 1-2 days |
| `portability:drill-check` | LOW | All | 7+ | 1-2 days |

---

## Cost-Benefit Analysis (HIGH Priority Skills)

This section provides cost-benefit analysis for HIGH priority skills to support investment decisions.

### `repo:workspace-check`

**Implementation Cost:**
- Development: 1-2 days (8-16 hours)
- Testing: 0.5 day (4 hours)
- Documentation: 0.25 day (2 hours)
- **Total: 1.75-2.75 days (14-22 hours)**

**Annual Maintenance Cost:** 2-4 hours (minor updates for new workspace types)

**Value Delivered:**
- **Risk Reduction:** Prevents workspace misconfiguration (estimated 3-4 incidents/year @ 2 days/incident = 6-8 days saved)
- **Automation:** Replaces manual workspace detection (estimated 0.5 hours/PR × 100 PRs/year = 50 hours saved)
- **Repository Enablement:** Unblocks hjps-ops and hjps-toolkit adoption (2 critical repositories)

**ROI Calculation:**
- **Cost:** 14-22 hours implementation + 2-4 hours/year maintenance = ~20 hours Year 1
- **Benefit:** 6-8 days (48-64 hours) incident prevention + 50 hours automation = 98-114 hours saved/year
- **ROI:** ~5:1 (conservative estimate)

**Priority Justification:** HIGH - Blocks 2 repositories, high ROI, foundational capability

---

### `repo:monorepo-check`

**Implementation Cost:**
- Development: 2-3 days (16-24 hours)
- Testing: 1 day (8 hours) - Complex due to Lerna/Changesets variations
- Documentation: 0.5 day (4 hours)
- **Total: 3.5-4.5 days (28-36 hours)**

**Annual Maintenance Cost:** 4-8 hours (updates for Lerna/Changesets changes)

**Value Delivered:**
- **Risk Reduction:** Prevents monorepo misconfiguration (estimated 2-3 incidents/year @ 3 days/incident = 6-9 days saved)
- **Quality Improvement:** Ensures proper versioning and publishing setup (prevents version conflicts)
- **Repository Enablement:** Unblocks hjps-toolkit adoption (critical shared library repository)

**ROI Calculation:**
- **Cost:** 28-36 hours implementation + 4-8 hours/year maintenance = ~36 hours Year 1
- **Benefit:** 6-9 days (48-72 hours) incident prevention + versioning quality improvements = 50-80 hours saved/year
- **ROI:** ~1.5:1 to 2:1

**Priority Justification:** HIGH - Unblocks hjps-toolkit which other repositories depend on

---

### `repo:language-check`

**Implementation Cost:**
- Development: 1-2 days (8-16 hours)
- Testing: 0.5 day (4 hours) - Test Python, Node, and hybrid detection
- Documentation: 0.25 day (2 hours)
- **Total: 1.75-2.75 days (14-22 hours)**

**Annual Maintenance Cost:** 2-4 hours (support for new languages if needed)

**Value Delivered:**
- **Risk Reduction:** Prevents multi-language validation failures (estimated 2-3 incidents/year @ 2 days/incident = 4-6 days saved)
- **Automation:** Automatic language stack detection (replaces manual checks)
- **Repository Enablement:** Unblocks hubspot-dataops adoption (first multi-language repo, template for future)
- **Strategic Value:** Enables tryton adoption in future (also multi-language)

**ROI Calculation:**
- **Cost:** 14-22 hours implementation + 2-4 hours/year maintenance = ~20 hours Year 1
- **Benefit:** 4-6 days (32-48 hours) incident prevention + enables 2+ multi-language repos = 40-60 hours saved/year
- **ROI:** ~2:1 to 3:1

**Priority Justification:** HIGH - First multi-language support, template for future repos

---

### `deps:file-protocol-check`

**Implementation Cost:**
- Development: 1-2 days (8-16 hours)
- Testing: 0.5 day (4 hours)
- Documentation: 0.25 day (2 hours)
- **Total: 1.75-2.75 days (14-22 hours)**

**Annual Maintenance Cost:** 1-2 hours (minimal changes expected)

**Value Delivered:**
- **Risk Reduction:** Prevents file: protocol CI failures (currently blocking 2 repositories)
- **Transparency:** Identifies repositories using file: dependencies (enables migration planning)
- **Migration Support:** Validates migration to registry-based dependencies

**ROI Calculation:**
- **Cost:** 14-22 hours implementation + 1-2 hours/year maintenance = ~18 hours Year 1
- **Benefit:** Unblocks 2 repositories immediately + prevents future file: protocol usage = 40+ hours saved
- **ROI:** ~2:1 (immediate unblocking value)

**Priority Justification:** HIGH - Blocks hjps-ops and hubspot-dataops adoption immediately

---

### `arch:module-system-check`

**Implementation Cost:**
- Development: 2-3 days (16-24 hours) - Complex due to CJS/ESM interop
- Testing: 1 day (8 hours) - Test mixed module systems
- Documentation: 0.5 day (4 hours)
- **Total: 3.5-4.5 days (28-36 hours)**

**Annual Maintenance Cost:** 4-6 hours (updates for module system changes)

**Value Delivered:**
- **Risk Reduction:** Prevents module system conflicts (estimated 1-2 incidents/year @ 3 days/incident = 3-6 days saved)
- **Documentation:** Explicit documentation of module system choices
- **Repository Enablement:** Unblocks hjps-ops (mixed CJS/ESM)

**ROI Calculation:**
- **Cost:** 28-36 hours implementation + 4-6 hours/year maintenance = ~34 hours Year 1
- **Benefit:** 3-6 days (24-48 hours) incident prevention + documentation value = 30-50 hours saved/year
- **ROI:** ~1:1 to 1.5:1

**Priority Justification:** HIGH - Necessary for hjps-ops adoption, prevents hard-to-debug module issues

---

### `perf:bundle-budget-check`

**Implementation Cost:**
- Development: 2-3 days (16-24 hours) - Integration with Next.js bundle analysis
- Testing: 0.5 day (4 hours)
- Documentation: 0.5 day (4 hours)
- **Total: 3-4 days (24-32 hours)**

**Annual Maintenance Cost:** 4-8 hours (updates for Next.js changes)

**Value Delivered:**
- **Risk Reduction:** Prevents bundle bloat (estimated 1-2 incidents/year @ 5 days/incident = 5-10 days saved)
- **Performance:** Maintains app performance (user experience value)
- **Automation:** Automated bundle size validation in CI (replaces manual checks)
- **Cost Savings:** Prevents CDN cost increases from bloated bundles

**ROI Calculation:**
- **Cost:** 24-32 hours implementation + 4-8 hours/year maintenance = ~32 hours Year 1
- **Benefit:** 5-10 days (40-80 hours) incident prevention + performance/cost savings = 50-90 hours saved/year
- **ROI:** ~1.5:1 to 3:1

**Priority Justification:** HIGH - Critical for hjps-frontend performance, user experience impact

---

### `api:openapi-check`

**Implementation Cost:**
- Development: 1-2 days (8-16 hours)
- Testing: 0.5 day (4 hours)
- Documentation: 0.25 day (2 hours)
- **Total: 1.75-2.75 days (14-22 hours)**

**Annual Maintenance Cost:** 2-4 hours (updates for OpenAPI spec changes)

**Value Delivered:**
- **Risk Reduction:** Prevents API client/server mismatches (estimated 2-3 incidents/year @ 2 days/incident = 4-6 days saved)
- **Quality:** Ensures type-safe API clients
- **Automation:** Automated OpenAPI client validation

**ROI Calculation:**
- **Cost:** 14-22 hours implementation + 2-4 hours/year maintenance = ~20 hours Year 1
- **Benefit:** 4-6 days (32-48 hours) incident prevention + type safety value = 35-50 hours saved/year
- **ROI:** ~1.75:1 to 2.5:1

**Priority Justification:** HIGH - Critical for hjps-frontend API integration quality

---

## Cost-Benefit Summary (HIGH Priority Skills)

| Skill | Implementation Cost | Maintenance Cost/Year | Value Delivered/Year | ROI | Strategic Value |
|-------|-------|---------|----------|-----|-----------------|
| `repo:workspace-check` | 14-22 hours | 2-4 hours | 98-114 hours | ~5:1 | Unblocks 2 repos |
| `repo:monorepo-check` | 28-36 hours | 4-8 hours | 50-80 hours | ~1.5-2:1 | Unblocks hjps-toolkit |
| `repo:language-check` | 14-22 hours | 2-4 hours | 40-60 hours | ~2-3:1 | Enables multi-language |
| `deps:file-protocol-check` | 14-22 hours | 1-2 hours | 40+ hours | ~2:1 | Immediate unblocking |
| `arch:module-system-check` | 28-36 hours | 4-6 hours | 30-50 hours | ~1-1.5:1 | Prevents hard bugs |
| `perf:bundle-budget-check` | 24-32 hours | 4-8 hours | 50-90 hours | ~1.5-3:1 | UX + cost savings |
| `api:openapi-check` | 14-22 hours | 2-4 hours | 35-50 hours | ~1.75-2.5:1 | Type safety |

**Total Investment (7 HIGH Priority Skills):**
- **Implementation:** 136-192 hours (17-24 days)
- **Annual Maintenance:** 19-36 hours (2.4-4.5 days)
- **Year 1 Total:** 155-228 hours (19.4-28.5 days)

**Total Value Delivered (Year 1):**
- **Quantified:** 343-494 hours (42.9-61.8 days) saved
- **Strategic:** Unblocks 4 repositories (hjps-ops, hjps-toolkit, hubspot-dataops, hjps-frontend)
- **Organizational:** Establishes patterns for Phase 7+ adoptions

**Overall ROI:** ~2.2:1 to 2.6:1 (conservative estimate, excluding strategic value)

**Investment Recommendation:** ✅ **APPROVE** - Strong ROI, unblocks critical repositories, enables organizational standards adoption

---

**See Also:**
- Phase 6 Priorities: `phase-6-priorities.md`
- Complete Review: `../../IMPLEMENTATION_PLAN_STANDARDS_REVIEW.md`
- Summary: `../SUMMARY.md`

