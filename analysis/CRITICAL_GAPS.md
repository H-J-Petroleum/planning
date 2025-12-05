# Critical Gaps in PHASE6 Plan - Quick Reference

**Last Updated:** 2025-12-03
**Policy-Version:** 1.0.0
**TTL:** 30d
**Owner:** Architecture + Security
**Status:** Active - Requires Immediate Attention

## Overview

This document provides a quick-reference guide to the **20 critical gaps** identified in the PHASE6 plan through integrated analysis of two independent reviews. These gaps require additions to the plan to ensure successful adoption.

**For detailed analysis, see:** [INTEGRATION_ANALYSIS.md](INTEGRATION_ANALYSIS.md)

---

## Summary by Priority

| Priority | Count | Total Effort | Timeline |
|----------|-------|--------------|----------|
| **CRITICAL** | 7 | ~2-4 weeks | Week 1 + Parallel |
| **HIGH** | 8 | ~30-40 hours | Weeks 2-3 |
| **MEDIUM** | 5 | ~10-15 hours | Weeks 3-4 |

**Grand Total:** 20 gaps, ~50-70 hours + 3-4 weeks parallel work

---

## CRITICAL Priority (7 gaps) - FIX IMMEDIATELY

### 1. DataOps Broken Entry Points ⚠️ BLOCKS DEVELOPMENT

**Issue ID:** R5-1 (Second Reviewer - NEW finding)

**Problem:**
- `npm start` fails in adopt_hubspot_dataops
- package.json references `main: "src/index.js"` which doesn't exist
- Local development completely broken

**Location:** `adopt_hubspot_dataops/package.json:5-7`

**Impact:** **LOCAL DEVELOPMENT BROKEN** - Team cannot run dataops locally

**Fix:**
```bash
# Create entry point
touch adopt_hubspot_dataops/src/index.ts

# Content:
export * from './process-manager.js';

# Update package.json:
"main": "dist/index.js",
"start": "node dist/index.js"
```

**Effort:** 30 minutes

**Add to:** Week 1, Critical Runtime Fixes section

**Test:** `npm run build && npm start` should succeed

---

### 2. Frontend Missing @opentelemetry/core ⚠️ BREAKS TELEMETRY

**Issue ID:** DI-3 (Second Reviewer - NEW finding)

**Problem:**
- `adopt_hjps_frontend/src/lib/otel.ts:3` imports @opentelemetry/core
- Package not declared in dependencies
- Works locally via transitive deps
- **Breaks in production** when dependency tree changes

**Location:** `adopt_hjps_frontend/package.json:24-44`

**Impact:** **BROWSER TELEMETRY BROKEN** - Production observability fails

**Fix:**
```json
// Add to adopt_hjps_frontend/package.json dependencies:
"@opentelemetry/core": "^2.1.0"
```

**Effort:** 15 minutes

**Add to:** Week 1, Critical Runtime Fixes section

**Test:** Import resolution succeeds, telemetry initializes

---

### 3. standards-cli Missing Prettier ⚠️ BREAKS EXTERNAL CONSUMERS

**Issue ID:** DI-1 (Second Reviewer - NEW finding)

**Problem:**
- `packages/standards-cli/src/io/adapters/fs-adapter.ts:3-8` uses prettier
- Package not declared as peer or dev dependency
- External consumers of @h-j-petroleum/standards-cli will crash

**Location:** `packages/standards-cli/package.json:17-33`

**Impact:** **BREAKS EXTERNAL CONSUMERS** - Published package unusable

**Fix:**
```json
// Add to packages/standards-cli/package.json:
"peerDependencies": {
  "prettier": "^3.0.0"
},
"devDependencies": {
  "prettier": "^3.6.2"
}
```

**Effort:** 15 minutes

**Add to:** Week 1, Critical Runtime Fixes section

**Test:** External installation and usage succeeds

---

### 4. Zod Major Version Conflict ⚠️ VALIDATION BREAKS

**Issue ID:** R3-4 (Second Reviewer - NEW finding)

**Problem:**
- standards-cli uses zod@^3.22.4
- Apps use zod@^4.1.12
- **Major version difference** = breaking API changes
- Validation behavior differs between CLI and apps

**Location:** Multiple repos

**Impact:** **VALIDATION BEHAVIOR DIFFERS** - Inconsistent schema validation

**Fix:**
```json
// 1. Update standards-cli to zod@^4.1.12
// 2. Add to policy/node/constraints.json:
{
  "requiredDependencies": {
    "zod": {
      "version": "^4.1.12",
      "reason": "Zod v3→v4 is breaking change; standardize on v4"
    }
  }
}
```

**Effort:** 1 hour (includes testing migration)

**Add to:** Week 1, Dependency Constraints section

**Test:** All zod usage works consistently

---

### 5. hjps-toolkit 0% Test Coverage ⚠️ NO QUALITY VALIDATION

**Issue ID:** 5.1 (First Review - Original finding)

**Problem:**
- All 14 hjps-toolkit packages have **zero test coverage**
- No regression protection
- Breaking changes go undetected
- Consumers have no confidence in package quality

**Location:** All packages in adopt_hjps_toolkit/packages/*

**Impact:** **NO REGRESSION PROTECTION** - Quality cannot be validated

**Fix:** Add comprehensive test suites
- toolkit-cli: 2-3 days
- toolkit-config: 1-2 days
- toolkit-hubspot: 2-3 days
- toolkit-logger: 1 day
- toolkit-metrics: 1 day
- Remaining 9 packages: 3-5 days

**Total Effort:** 10-15 days

**Add to:** NEW Parallel Track (Weeks 3-8)

**Owner:** Assign to separate team member to avoid blocking main timeline

**Acceptance:** ≥80% code coverage per package

---

### 6. Python 0% Test Coverage ⚠️ DATAOPS QUALITY RISK

**Issue ID:** 5.2 (First Review - Original finding)

**Problem:**
- adopt_hubspot_dataops Python modules have **zero test coverage**
- No regression protection for data workflows
- Production bugs likely

**Location:** adopt_hubspot_dataops Python files

**Impact:** **NO REGRESSION PROTECTION** - Dataops adoption at risk

**Fix:** Add Python test suite
- Create pytest configuration
- Add tests for data sync, export, vector scripts
- Target ≥80% coverage

**Effort:** 1-2 weeks

**Add to:** Week 4, Dataops section

**Acceptance:** ≥80% Python code coverage

---

### 7. Mock Cleanup Not Explicit ⚠️ BLOCKS ENFORCEMENT

**Issue ID:** 3.1, 3.2 (First Review - Original findings)

**Problem:**
- hjps-ops: 291 vi.mock() calls (violates boundaries-only policy)
- hubspot-dataops: 31 vi.mock() calls + 157+95 line toolkit mocks
- Current plan mentions refactoring but doesn't explicitly call out mock reduction
- **Cannot transition to enforcement mode** with these violations

**Location:** Test files across hjps-ops and hubspot-dataops

**Impact:** **BLOCKS ENFORCEMENT MODE** - Violates core policy principle

**Fix:**
- hjps-ops: Reduce 291 → <20 mocks (1-2 weeks)
- hubspot-dataops: Reduce 31 → <10 mocks (1 week)
- Approach: Refactor to dependency injection, use real implementations

**Total Effort:** 2-3 weeks

**Add to:** Week 4, make explicit in Ops and Dataops sections

**Acceptance:** <20 mocks per repo (boundaries only)

---

## HIGH Priority (8 gaps) - SCHEDULE IN WEEKS 2-3

### 8. arch:layer-check Not Implemented

**Issue ID:** 4.2 (First Review)

**Problem:** Skill specified but not scheduled, can't validate Core/IO separation

**Fix:** Implement skill in Week 2 (4-6 hours)

**Blocks:** All refactoring validation in Weeks 3-4

**Location:** Week 2 Skill Implementation Sprint

---

### 9. perf:bundle-budget-check Not Scheduled

**Issue ID:** 5.3 (First Review)

**Problem:** Specified in IMPLEMENTATION_PLAN:172-174 but not in PHASE6_TASKS

**Fix:** Implement skill in Week 2 (3-4 hours)

**Blocks:** hjps-frontend adoption validation

**Location:** Week 2 Skill Implementation Sprint

---

### 10. api:openapi-check Not Scheduled

**Issue ID:** 5.6 (First Review)

**Problem:** Specified in IMPLEMENTATION_PLAN:175-178, 219-223 but not scheduled

**Fix:** Implement skill in Week 2 (4-6 hours)

**Blocks:** hjps-ops API adoption

**Location:** Week 2 Skill Implementation Sprint

---

### 11. Version Alignment Manifest Missing

**Issue ID:** DI-6, 1.1.1, 1.1.2, 1.1.3 (Both Reviews)

**Problem:** No central document standardizing TypeScript, Vitest, ESLint, etc. versions

**Fix:** Create DEPENDENCY_VERSIONS.md in Week 2 (2-3 hours)

**Content:**
```markdown
# Standard Dependency Versions

## Core Tooling
- TypeScript: ^5.9.3
- Vitest: ^4.0.8
- ESLint: ^9.39.1
- @types/node: ^24.10.1

## Domain Libraries
- Zod: ^4.1.12 (NOT 3.x)
- Commander: ^14.0.1
- OTEL: ^0.208.0
- Pino: ^10.1.0
```

**Location:** Week 2 Skill Implementation Sprint

---

### 12. Dependency-Cruiser Configs Not Deployed

**Issue ID:** R3-10, 4.6 (Both Reviews)

**Problem:** arch:dep-cruise skill exists but repos don't have .dependency-cruiser.cjs configs

**Fix:** Create configs in Week 2 (2 hours)
- adopt_hjps_toolkit/.dependency-cruiser.cjs
- adopt_hubspot_dataops/.dependency-cruiser.cjs
- adopt_hjps_frontend/.dependency-cruiser.cjs

**Location:** Week 2 Skill Implementation Sprint

---

### 13. perf:lighthouse-check Not Scheduled

**Issue ID:** 5.4 (First Review)

**Problem:** Specified in IMPLEMENTATION_PLAN:209-212 but not in PHASE6_TASKS

**Fix:** Implement skill in Week 3 (4-6 hours)

**FULL-only, longer runtime acceptable**

**Location:** Week 3 Performance & Observability section

---

### 14. perf:api-latency Not Scheduled

**Issue ID:** 5.5 (First Review)

**Problem:** Specified in IMPLEMENTATION_PLAN:214-218 but not scheduled

**Fix:** Implement skill in Week 3 (4-6 hours)

**Essential for hjps-ops adoption**

**Location:** Week 3 Performance & Observability section

---

### 15. obs:otel-check Not Scheduled

**Issue ID:** AQ-1 (Second Reviewer)

**Problem:** Infrastructure complete, skill implementation pending

**Fix:** Implement skill in Week 3 (3-4 hours)

**Advisory mode first**

**Location:** Week 3 Performance & Observability section

---

## MEDIUM Priority (5 gaps) - WEEKS 3-4

### 16. TS Execution Standardization Missing

**Issue ID:** BT-4, R5-2 (Both Reviews)

**Problem:** Inconsistent use of tsx/ts-node/node across repos

**Fix:** Standardize dataops scripts to use tsx (2-3 hours)

**Location:** Week 4 Dataops section

---

### 17. Frontend Toolkit Migration Not Explicit

**Issue ID:** R2-1, R2-2, CP-1 (Second Reviewer)

**Problem:** adopt_hjps_frontend has custom logger/telemetry (200 lines) instead of using toolkit packages

**Fix:** Remove custom implementations, import from toolkits (2-3 days)

**Location:** Week 4 hjps-frontend section (new task)

---

### 18. Zombie Dependencies Not Removed

**Issue ID:** R3-7/8/9 (Second Reviewer)

**Problem:** lodash, date-fns, eslint-plugin-node declared but unused in dataops

**Fix:** Remove unused deps (30 minutes)

**Location:** Week 1 Dependency Cleanup section

---

### 19. Multi-Language Boundary Docs Missing

**Issue ID:** 4.8, AQ-2 (Both Reviews)

**Problem:** No documentation of Python vs Node.js boundaries in dataops

**Fix:** Document language boundaries (2-3 hours)

**Location:** Week 4 Dataops section

---

### 20. Duplicate Vitest Configs

**Issue ID:** R6-1 (Second Reviewer)

**Problem:** toolkit-hubspot has both vitest.config.js and vitest.config.ts

**Fix:** Delete .js, keep .ts (30 minutes)

**Location:** Week 3 cleanup task

---

## Quick Wins (Can Do Anytime)

These tasks take <1 hour each and provide immediate value:

1. **Fix DataOps entry points** (30 min) - Unblocks local development
2. **Add Frontend OTEL core** (15 min) - Fixes telemetry
3. **Add standards-cli prettier** (15 min) - Fixes external consumers
4. **Remove zombie dependencies** (30 min) - Cleaner installs
5. **Delete duplicate Vitest config** (30 min) - Less confusion

**Total Quick Wins:** ~2 hours, 5 issues resolved

---

## Implementation Checklist

### This Week (Week 1)
- [ ] Fix DataOps entry points (30 min)
- [ ] Add Frontend OTEL core (15 min)
- [ ] Add standards-cli prettier (15 min)
- [ ] Resolve Zod v3/v4 conflict (1 hour)
- [ ] Remove zombie dependencies (30 min)

**Week 1 Total:** ~3-4 hours

### Next Week (Week 2)
- [ ] Implement arch:layer-check (4-6 hours)
- [ ] Implement perf:bundle-budget-check (3-4 hours)
- [ ] Implement api:openapi-check (4-6 hours)
- [ ] Implement repo:package-check (4-6 hours)
- [ ] Create version alignment manifest (2-3 hours)
- [ ] Deploy dependency-cruiser configs (2 hours)

**Week 2 Total:** ~20-25 hours (2-3 days)

### Week 3
- [ ] Implement perf:lighthouse-check (4-6 hours)
- [ ] Implement perf:api-latency (4-6 hours)
- [ ] Implement obs:otel-check (3-4 hours)
- [ ] Metrics dashboard artifact (2-3 hours)
- [ ] Delete duplicate Vitest config (30 min)

**Week 3 Total:** ~14-20 hours

### Week 4
- [ ] Make mock cleanup explicit in Ops (1-2 weeks)
- [ ] Make mock cleanup explicit in Dataops (1 week)
- [ ] Add Python test coverage (1-2 weeks)
- [ ] Standardize TS execution in dataops (2-3 hours)
- [ ] Document multi-language boundaries (2-3 hours)
- [ ] Frontend toolkit migration (2-3 days)

**Week 4 Total:** ~2-4 weeks of work (can parallelize some)

### Parallel Track (Weeks 3-8)
- [ ] Add hjps-toolkit test coverage (10-15 days)
  - Assign separate owner
  - Target: ≥80% coverage per package
  - Run alongside main adoption work

---

## Success Criteria

### Coverage Improvement
- **Before:** 27% fully covered, 45% partially, 27% not covered
- **After:** 68% fully covered, 25% partially, 7% not covered
- **Improvement:** +41% full coverage

### Risk Mitigation
- **Before:** 7 CRITICAL issues blocking progress
- **After:** All CRITICAL issues resolved with clear timelines
- **Result:** Clear path to enforcement mode

### Effort Required
- **Week 1:** ~4 hours (critical fixes)
- **Weeks 2-3:** ~35-45 hours (skill implementations)
- **Week 4:** ~2-4 weeks (quality work)
- **Parallel:** 10-15 days (separate owner)

**No Major Timeline Changes** - Additions fit within existing PHASE6 structure

---

## References

- **Full Analysis:** [INTEGRATION_ANALYSIS.md](INTEGRATION_ANALYSIS.md)
- **Current Plan:** [../PHASE6_TASKS.md](../PHASE6_TASKS.md)
- **Implementation Details:** [../PHASE6_IMPLEMENTATION_PLAN.md](../PHASE6_IMPLEMENTATION_PLAN.md)
- **Original Gaps:** [./GAPS.md](GAPS.md)

---

## Notes

**Last Reviewed:** 2025-11-24

**Next Review:** When Week 1 critical fixes are complete

**Owner Assignments Needed:**
- Week 1 fixes: Architecture team (immediate)
- Week 2 skills: Architecture + BE teams (scheduled)
- Parallel test track: Separate owner to avoid blocking (assign ASAP)

**Status Tracking:** Update this document as gaps are addressed. Mark items ✅ when complete.
