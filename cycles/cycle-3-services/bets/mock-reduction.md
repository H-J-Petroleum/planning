# Bet: Mock Reduction Campaign

**Status**: ![Status: Proposed](https://img.shields.io/badge/status%2Fproposed-d4c5f9)
**Cycle**: 3
**Owner**: Ops

<!-- LABEL SYSTEM METADATA -->
![Priority: Medium](https://img.shields.io/badge/priority%2Fmedium-ffc107)
![Type: Cleanup](https://img.shields.io/badge/type%2Fcleanup-d4c5f9)
![Scope: Multi Repo](https://img.shields.io/badge/scope%2Fmulti--repo-7057ff)
![Effort: 13](https://img.shields.io/badge/effort%2F13-2a7515)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

**Goal**: [Phase 6 Adoption](../../../goals/phase-6-adoption.md)

---

## 1. Shaping (Planning)

### Context & Problem
We have too many mocks hiding bugs. We need to reduce them significantly to improve test reliability and confidence.

### Scope
- **In Scope**: `hjps-ops` and `dataops` test suites.
- **Out of Scope**: `toolkit` tests (already covered).

### Research & Options
- **Decision**: Use `testcontainers` for integration tests where possible.

### Solution Design
Replace mocks with real dependencies or lightweight fakes.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Reduce `hjps-ops` Mocks**
  - [ ] Reduce 291 mocks to <20
  - [ ] Use `pg-mem` or testcontainers
- [ ] **Reduce `dataops` Mocks**
  - [ ] Reduce 31 mocks to <10
  - [ ] Use real HubSpot sandbox for integration tests

### Acceptance Criteria
- [ ] `hjps-ops` mock count < 20
- [ ] `dataops` mock count < 10
- [ ] Test execution time remains acceptable

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `hjps-ops/test/`
- `dataops/test/`

### References
- [Testing Strategy](../../../TESTING_STRATEGY.md)

### Notes for AI
Prioritize high-value tests.

---

## 3. Review (Cool-Down)

### Verification
- [ ] Run tests -> Pass
- [ ] Count mocks -> Below threshold

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
