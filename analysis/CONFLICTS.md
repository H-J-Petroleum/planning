# Critical Conflicts Analysis

**Last Updated:** 2025-11-11  
**Policy-Version:** 1.0.0  
**TTL:** 90d  
**Owner:** Architecture + Security

---

## Overview

This document identifies critical conflicts between documented standards and actual implementation.

---

## 🔴 Critical Conflicts

### 1. Testing Policy: No-Mocks vs. Actual Implementation

**Conflict:**
- **AGENTIC_IMPLEMENTATION_PLAN.md** (Section 7, 10): States strict no-mocks policy, ESLint bans `vi.mock()` and `jest.mock()`
- **DEPENDENCY_STANDARDS_2025.md** (Section 7): References no-mocks policy
- **AGENTS.md** (Section Testing): Enforces no-mocks policy
- **Reality:** Many test files use `vi.mock()` extensively:
  - `scripts/skills/__tests__/security-secret-scan.test.ts` - Uses `vi.mock("node:child_process")`
  - `scripts/skills/__tests__/sbom-generate.test.ts` - Uses multiple `vi.mock()` calls
  - `scripts/skills/__tests__/test-coverage.test.ts` - Uses `vi.mock()` for utilities

**Impact:** HIGH - Policy is documented but not enforced in practice

**Recommendation:**
1. **Option A (Preferred):** Update implementation plan to reflect current reality - allow mocks for external boundaries (child_process, fs, network) but require DI for business logic
2. **Option B:** Enforce no-mocks policy via ESLint rule and migrate all tests to DI pattern
3. **Option C:** Clarify policy - no mocks in `src/logic/**` but allow mocks for adapter/boundary testing

**Action Required:** Update `docs/AGENTIC_IMPLEMENTATION_PLAN.md` Section 7 and 10 to clarify testing policy scope

---

## Detailed Analysis

### Testing Policy Deep Dive

**Current State:**
- Policy says: "ESLint rule bans `vi.mock()` and `jest.mock()`"
- Reality: Many test files use `vi.mock()` for:
  - `node:child_process` (external boundary)
  - `node:fs` (external boundary)
  - Utility modules (internal boundaries)

**Analysis:**
The no-mocks policy makes sense for `src/logic/**` (business logic should be pure and testable via DI). However, for adapter/boundary testing (child_process, fs, network), mocks are reasonable.

**Recommendation:**
Update policy to:
- ✅ **Enforce no-mocks** in `src/logic/**` (business logic)
- ✅ **Allow mocks** in `src/adapters/**` tests (external boundaries)
- ✅ **Require DI** for all business logic dependencies
- ✅ **Update ESLint rule** to only ban mocks in logic layer

---

**See Also:**
- Complete Review: `../IMPLEMENTATION_PLAN_STANDARDS_REVIEW.md` (Section: Critical Conflicts)
- Summary: `SUMMARY.md`
- Recommendations: `recommendations/phase-6-priorities.md`

