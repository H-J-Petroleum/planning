# Implementation Plan Standards Analysis - Executive Summary

**Last Updated:** 2025-11-11  
**Policy-Version:** 1.0.0  
**TTL:** 90d  
**Owner:** Architecture + Security

---

## Overview

This analysis reviews `docs/AGENTIC_IMPLEMENTATION_PLAN.md` against all standards, policies, and guidelines across the organization to identify conflicts, gaps, and adoption requirements.

**Status:** Phase 5 Complete - Reviewing for Phase 6+ improvements

---

## Key Findings

### 🔴 Critical Conflicts (0 Active, 1 Resolved)

1. **✅ RESOLVED: Testing Policy Conflict** (2025-11-11)
   - Plan stated strict no-mocks policy
   - Reality: 8.8% of tests use `vi.mock()` for boundaries
   - **Resolution:** Clarified as "Boundaries-Only Mocking Policy" - mocking allowed for system boundaries only (`node:fs`, `node:child_process`), forbidden for business logic

### ⚠️ Standards Not Captured (8)

1. **Performance Standards** - PERF_STANDARDS.md SLOs not enforced
2. **Cost Guards** - COST_GUARDS.md budgets not automated
3. **Observability** - OBSERVABILITY.md requirements not validated
4. **Portability** - PORTABILITY.md quarterly drills not enforced
5. **Security Gaps** - OIDC/PKCE, webhook signatures, RBAC not validated
6. **Dependency Gaps** - Framework versions (React 19, Next 15.5.4) not enforced
7. **Governance** - Branch naming, PR requirements not enforced
8. **Documentation TTL** - TTL header validation could be enhanced

### ✅ Well-Covered Areas

- Architecture rails (dep-cruise, purity-scan)
- Basic dependency governance (banned packages, licenses)
- Documentation hygiene (balance, freshness)
- Repository hygiene (script-map, hygiene)
- Security scanning (vulnerability, secrets, SBOM)

---

## Repository Adoption Status

| Repository | Workspace | Module System | Blockers | Readiness |
|------------|-----------|--------------|----------|-----------|
| **hjps-ops** | npm workspaces | Mixed (CJS+ESM) | Workspace support, file: deps | ⚠️ Medium |
| **hjps-toolkit** | npm + Lerna | ESM-only | Monorepo support, per-package | ⚠️ Medium |
| **hubspot-dataops** | npm (standalone) | Python/Node hybrid | Multi-language support, file: deps (6 packages) | ⚠️ Medium |
| **hjps-frontend** | npm (no workspaces) | ESM (Bundler) | Bundle budgets, OpenAPI, Next.js | ⚠️ Medium |
| **vector-toolkit** | pnpm workspaces | ESM (Bundler) | Node version, nested packages, embedding costs | ⚠️ Medium |
| **tryton** | Unknown (not created) | Python/PostgreSQL | Multi-language, ERP patterns, dual-write | ⚠️ Low |
| **toolkit-standards** | pnpm | ESM | None | ✅ Ready |

**Note:** hubspot-dataops status updated from "Unknown" to "npm (standalone)" based on validated implementation analysis (2025-11-11). Python/Node hybrid validated with specific dependencies. See `repositories/hubspot-dataops.md` for details.

---

## Inter-Repository Dependencies

### Dependency Graph

```
toolkit-standards (reference)
    │
    ├──> hjps-toolkit (monorepo - 10+ packages)
    │       ├── @hjps/toolkit-core
    │       ├── @hjps/toolkit-config
    │       ├── @hjps/toolkit-cli
    │       ├── @hjps/toolkit-hubspot
    │       ├── @hjps/toolkit-logger
    │       ├── @hjps/toolkit-metrics
    │       └── ... (6 more packages)
    │
    ├──> hjps-ops
    │       └──> hjps-toolkit (file: protocol - 6+ packages)
    │
    ├──> hubspot-dataops
    │       └──> hjps-toolkit (file: protocol - 6 packages):
    │               - @hjps/toolkit-cli
    │               - @hjps/toolkit-config
    │               - @hjps/toolkit-hubspot
    │               - @hjps/toolkit-logger
    │               - @hjps/toolkit-metrics
    │               - @hjps/toolkit-testing (devDep)
    │
    ├──> hjps-frontend
    │       └──> (no hjps-toolkit dependencies)
    │
    └──> vector-toolkit (planned)
            └──> (planned for hubspot-dataops integration)
```

### Dependency Issues

**file: Protocol Dependencies (HIGH Priority Blocker):**
- **Affected:** hjps-ops, hubspot-dataops
- **Issue:** Hardcoded directory structure (`file:../hjps-toolkit/packages/*`)
- **Impact:** Breaks standard CI/CD without special setup
- **Solution:** Migrate to private registry (GitHub Packages) or monorepo root
- **See:** `docs/DEPENDENCY_STRATEGY.md`

### Adoption Sequence

**Recommended Order (Based on Dependencies):**

```
Phase 6.1: Foundation (Weeks 1-2)
├── toolkit-standards ✅ (reference implementation)
├── hjps-toolkit (publish packages to registry)
└── Resolve file: protocol dependencies

Phase 6.2: Core Infrastructure (Weeks 3-4)
├── hjps-ops (depends on hjps-toolkit)
└── hjps-toolkit (monorepo support)

Phase 6.3: Applications (Weeks 5-6, can run in parallel)
├── hjps-frontend (independent)
├── hubspot-dataops (depends on hjps-toolkit)
└── vector-toolkit (planned integration with hubspot-dataops)

Phase 7+: Future
└── tryton (depends on hubspot-dataops, not yet created)
```

**Rationale:**
1. **hjps-toolkit first** - Other repos depend on it (hjps-ops, hubspot-dataops)
2. **Resolve file: protocol** - Blocks hjps-ops and hubspot-dataops adoption
3. **hjps-frontend parallel** - No dependencies, can adopt independently
4. **vector-toolkit after Phase 6** - Integration with hubspot-dataops planned for Week 8
5. **tryton last** - Phase 4+ repository, depends on hubspot-dataops infrastructure

---

## Business Priority Rationale

**Priority is based on:**
1. **Dependency Order** - hjps-toolkit blocks hubspot-dataops and hjps-ops
2. **Organizational Impact** - hjps-ops is core infrastructure
3. **Team Readiness** - Frontend team ready, Python repos need prep
4. **Risk Reduction** - Start with simpler repos to validate process

**No Hard Deadlines:**
- Timeline is effort-based with flexibility
- No contractual or compliance deadlines identified
- Internal engineering standards initiative

**Business Context:**
- No revenue impact analysis documented
- No customer-facing deadline pressures
- Focus on technical debt reduction and standardization
- Enables future scalability and maintainability

---

## Priority Actions

### High Priority (Phase 5.5 or Phase 6)

1. **✅ COMPLETED: Resolve Testing Policy Conflict** (2025-11-11)
2. **Resolve file: Protocol Dependencies** (hjps-ops, hubspot-dataops) - See `docs/DEPENDENCY_STRATEGY.md`
3. **Add Workspace Support** (npm workspaces)
4. **Create Reusable Policy-Checks Workflow**
5. **Enhance Dependency Policy** (framework versions)

### Medium Priority (Phase 6)

6. **Add Performance Skills**
7. **Add hjps-ops Specific Skills**
8. **Add hjps-toolkit Specific Skills**
9. **Add hubspot-dataops Specific Skills**
10. **Add hjps-frontend Specific Skills**
11. **Add vector-toolkit Specific Skills**
12. **Add Tryton ERP Specific Skills** (Phase 4+)

### Low Priority (Phase 7+)

13. **Add Cost Guard Skills**
14. **Add Observability Skills**
15. **Add Portability Skills**
16. **Add Governance Skills**

---

## Overall Risk Assessment

### Organizational Adoption Risk Summary

| Repository | Overall Risk | Primary Risks | Readiness for Phase 6 |
|------------|--------------|---------------|----------------------|
| **toolkit-standards** | ✅ **LOW** | None - reference implementation | ✅ Ready Now |
| **hjps-ops** | ⚠️ **MEDIUM-HIGH** | file: protocol, workspace support | After skills implementation |
| **hjps-toolkit** | ⚠️ **MEDIUM** | Monorepo complexity, per-package validation | After skills implementation |
| **hjps-frontend** | ✅ **LOW-MEDIUM** | Bundle budgets, Next.js patterns | Ready with frontend skills |
| **hubspot-dataops** | 🔴 **HIGH** | Multi-language complexity, file: protocol | After multi-language support |
| **vector-toolkit** | ✅ **LOW-MEDIUM** | Node version (quick fix), nested workspaces | After workspace support |
| **tryton** | ⚠️ **MEDIUM-HIGH** | Not created yet, multi-language, ERP patterns | Phase 7+ |

### Critical Risk Factors Affecting Multiple Repositories

**1. file: Protocol Dependencies (CRITICAL - Affects 2 repos)**
- **Affected:** hjps-ops, hubspot-dataops
- **Impact:** Blocks CI/CD adoption
- **Mitigation:** See [DEPENDENCY_STRATEGY.md](../../architecture/DEPENDENCY_STRATEGY.md)
- **Priority:** HIGH (must resolve in Phase 6.1)
- **Effort:** 1-2 days per repository (after registry setup)

**2. Workspace/Monorepo Support (HIGH - Affects 3 repos)**
- **Affected:** hjps-ops (npm workspaces), hjps-toolkit (Lerna), vector-toolkit (pnpm nested)
- **Impact:** Cannot validate workspace repositories
- **Mitigation:** Implement `repo:workspace-check` and `repo:package-check` skills
- **Priority:** HIGH (Phase 6.1)
- **Effort:** 3-5 days total

**3. Multi-Language Support (HIGH - Affects 2+ repos)**
- **Affected:** hubspot-dataops (Python/Node), tryton (Python/PostgreSQL, future)
- **Impact:** Cannot validate Python dependencies
- **Mitigation:** Enhance `deps:policy-python`, implement `repo:language-check`
- **Priority:** MEDIUM (Phase 6.2)
- **Effort:** 3-4 days

### Risk Mitigation Strategy

**Phase 6.1 (Weeks 1-2): Address Critical Risks**
- ✅ Resolve file: protocol dependencies (publish hjps-toolkit to registry)
- ✅ Implement workspace support (npm/pnpm/Lerna)
- ✅ Create reusable policy-checks workflow

**Phase 6.2 (Weeks 3-4): Repository-Specific Risks**
- ✅ Add monorepo support (hjps-toolkit)
- ✅ Add multi-language support (hubspot-dataops)
- ✅ Add frontend-specific skills (hjps-frontend)

**Phase 6.3 (Weeks 5-6): Lower-Risk Adoptions**
- ✅ hjps-frontend adoption (lowest risk)
- ✅ vector-toolkit adoption (after Node version fix)
- ✅ Monitor and stabilize

**Phase 7+: Future Repositories**
- tryton adoption (after multi-language patterns validated)
- Additional repositories as needed

### Success Probability by Repository

| Repository | Success Probability | Confidence | Key Dependencies |
|------------|-------------------|------------|------------------|
| **toolkit-standards** | 100% | HIGH | Already adopted |
| **hjps-frontend** | 85% | MEDIUM | Bundle budgets, Next.js skills |
| **vector-toolkit** | 80% | MEDIUM | Node version fix, workspace support |
| **hjps-toolkit** | 75% | MEDIUM | Monorepo support, per-package validation |
| **hjps-ops** | 70% | MEDIUM-LOW | file: protocol, workspace support, mixed modules |
| **hubspot-dataops** | 60% | LOW | Multi-language, file: protocol |
| **tryton** | 50% | LOW | Not created, multi-language, ERP complexity |

**Overall Organizational Success Probability: 75%**
- **Rationale:** Critical risks are addressable, but require upfront skill implementation
- **Confidence:** MEDIUM - Based on Phase 0-5 implementation experience
- **Recommendation:** Address critical risks before beginning repository adoptions

---

## Detailed Analysis

- **Conflicts:** See `CONFLICTS.md`
- **Gaps:** See `GAPS.md`
- **Repository Requirements:** See `repositories/` directory
- **Recommendations:** See `recommendations/` directory
- **Complete Review:** See `../IMPLEMENTATION_PLAN_STANDARDS_REVIEW.md`

---

**Next Steps:** Review repository-specific analysis files and Phase 6 priorities to plan implementation.

