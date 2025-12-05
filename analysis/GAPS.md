# Standards Gaps Analysis

**Last Updated:** 2025-11-11  
**Policy-Version:** 1.0.0  
**TTL:** 90d  
**Owner:** Architecture + Security

---

## Overview

This document identifies standards documented in the organization that are not captured or enforced by the implementation plan.

---

## ⚠️ Standards Not Captured in Implementation Plan

### 2. Performance Standards (PERF_STANDARDS.md)

**Missing from Plan:**
- Frontend SLOs: TTFB p95 < 200ms (static), < 350ms (SSR)
- Web Vitals: INP p75 < 200ms, LCP p75 < 2.5s, CLS p75 < 0.1
- Bundle budgets: 150KB gzipped initial route, 50KB per chunk
- API SLOs: Reads p95 < 250ms, Writes p95 < 500ms
- Database SLOs: OLTP read p95 < 50-150ms, write p95 < 100-250ms
- CI/CD performance gates: Lighthouse perf ≥ 0.90, bundle diff alerts

**Impact:** MEDIUM - Performance standards exist but aren't enforced by policy system

**Recommendation:**
- Add `perf:budget-check` skill to validate bundle sizes
- Add `perf:lighthouse-check` skill for CI/CD gates
- Add `perf:api-latency` skill for API SLO validation
- Document in Phase 6 or Phase 7 as "Performance Enforcement"

**Action Required:** Add performance skills to implementation plan Phase 6+

---

### 3. Cost Guards & SLOs (COST_GUARDS.md)

**Missing from Plan:**
- API p95 latency budgets (reads < 250ms, writes < 500ms)
- Data freshness SLOs (p95 < 5min)
- PR time-to-green SLOs (< 60min small, < 24h bulk)
- Cost tripwires (CDN egress, serverless CPU, image bandwidth)
- Embeddings budget and concurrency caps

**Impact:** MEDIUM - Cost controls exist but aren't automated

**Recommendation:**
- Add `cost:budget-check` skill for cost tripwire validation
- Add `cost:embedding-budget` skill for embeddings monitoring
- Integrate with metrics collection system

**Action Required:** Add cost guard skills to implementation plan Phase 6+

---

### 4. Observability Standards (OBSERVABILITY.md)

**Missing from Plan:**
- Required OTel attributes: trace_id, request_id, user_id, pipeline_id, batch_id, duration_ms
- Trace propagation requirements (traceparent from FE)
- Dashboard requirements (latency/error rates, sync lag, embeddings throughput, cost)

**Impact:** LOW - Observability is operational, not policy-enforced

**Recommendation:**
- Add `obs:otel-check` skill to validate OTel instrumentation
- Add `obs:attribute-check` skill to validate required attributes
- Document as optional skill (advisory mode)

**Action Required:** Add observability validation skills to implementation plan Phase 6+

---

### 5. Portability Drills (PORTABILITY.md)

**Missing from Plan:**
- Quarterly database provider swap drills
- Quarterly frontend host swap drills
- Quarterly DuckDB Parquet export drills
- Evidence storage requirements (`docs/portability-reports/YYYY-MM/`)

**Impact:** LOW - Portability is operational, not policy-enforced

**Recommendation:**
- Add `portability:drill-check` skill to validate quarterly drills completed
- Add `portability:evidence-check` skill to validate evidence stored
- Document as optional skill (advisory mode)

**Action Required:** Add portability validation skills to implementation plan Phase 6+

---

### 6. Security Standards (SECURITY.md)

**Partially Captured:**
- ✅ Secret scanning: Covered by `security:secret-scan` skill
- ✅ SBOM: Covered by `sbom:generate` and `sbom:policy-eval` skills
- ❌ OIDC/PKCE: Not validated by policy system
- ❌ Webhook signature verification: Not validated by policy system
- ❌ RBAC claims validation: Not validated by policy system

**Impact:** MEDIUM - Some security standards aren't policy-enforced

**Recommendation:**
- Add `security:auth-check` skill to validate OIDC/PKCE implementation (advisory)
- Add `security:webhook-check` skill to validate signature verification (advisory)
- Add `security:rbac-check` skill to validate RBAC claims (advisory)

**Action Required:** Add missing security validation skills to implementation plan Phase 6+

---

### 7. Dependency Standards (DEPENDENCY_STANDARDS_2025.md)

**Partially Captured:**
- ✅ Node 24 requirement: Covered by `deps:policy-node` skill
- ✅ pnpm 9 requirement: Covered by `deps:policy-node` skill
- ✅ Banned packages (winston, jest, joi): Should be in `policy/node/constraints.json`
- ❌ Specific version requirements (React 19, Next 15.5.4, etc.): Not enforced
- ❌ ESLint v9 flat config requirement: Not validated
- ❌ Vitest requirement (not jest): Not validated

**Impact:** MEDIUM - Specific version standards aren't enforced

**Recommendation:**
- Enhance `deps:policy-node` skill to validate framework versions
- Add `deps:framework-check` skill to validate React/Next/Express versions
- Add `deps:tooling-check` skill to validate ESLint/Vitest usage
- Update `policy/node/constraints.json` with version floors for major frameworks

**Action Required:** Enhance dependency policy skills in Phase 6

---

### 8. Branching & PR Policy (BRANCHING_PR_POLICY.md, GOVERNANCE.md)

**Partially Captured:**
- ✅ Branch naming conventions: Not enforced by policy system
- ✅ PR requirements: Not enforced by policy system
- ✅ Squash merge requirement: Not enforced by policy system
- ✅ Code owner requirements: Not enforced by policy system

**Impact:** LOW - Governance is operational, not policy-enforced

**Recommendation:**
- Add `repo:branch-naming` skill to validate branch names (advisory)
- Add `repo:pr-checklist` skill to validate PR requirements (advisory)
- Document as optional skills for repositories that want governance enforcement

**Action Required:** Add governance validation skills to implementation plan Phase 6+ (optional)

---

### 9. Documentation TTL Requirements

**Inconsistency:**
- **AGENTIC_IMPLEMENTATION_PLAN.md** (Section 8): TTL defaults 90 days for README, ARCHITECTURE, CONTRIBUTING, SECURITY
- **Reality:** Some docs have TTL, some don't
- **POLICY_REFERENCE.md**: Has TTL header but not all docs follow pattern

**Impact:** LOW - Documentation standards exist but aren't consistently applied

**Recommendation:**
- `docs:freshness` skill should validate TTL headers
- Add TTL validation to all required docs
- Update enforcement mode to enforce TTL compliance

**Action Required:** Enhance `docs:freshness` skill to validate TTL headers

---

**See Also:**
- Complete Review: `../IMPLEMENTATION_PLAN_STANDARDS_REVIEW.md` (Section: Standards Not Captured)
- Summary: `SUMMARY.md`
- Recommendations: `recommendations/phase-6-priorities.md`

