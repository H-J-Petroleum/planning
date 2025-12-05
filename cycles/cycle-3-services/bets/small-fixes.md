# Bet: Small Fixes Batch

**Status**: ![Status: Proposed](https://img.shields.io/badge/status%2Fproposed-d4c5f9)
**Cycle**: 3
**Owner**: Ops

<!-- LABEL SYSTEM METADATA -->
![Priority: Low](https://img.shields.io/badge/priority%2Flow-8bc34a)
![Type: Cleanup](https://img.shields.io/badge/type%2Fcleanup-d4c5f9)
![Scope: Multi Repo](https://img.shields.io/badge/scope%2Fmulti--repo-7057ff)
![Effort: 1](https://img.shields.io/badge/effort%2F1-c2e0c6)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

**Goal**: [Phase 6 Adoption](../../../goals/phase-6-adoption.md)

---

## 1. Shaping (Planning)

### Context & Problem
Small fixes for Cycle 3, including performance checks and cross-cutting type safety.

### Scope
- **In Scope**: Listed tasks.

### Research & Options
- **Decision**: Batch execution.

### Solution Design
Checklist execution.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **[Ops] Migrate `pdf-generator` to ESM**
- [ ] **[Ops] Standardize TS Execution Pattern (`tsx`)**
- [ ] **[BE] Add `perf:api-latency`**
  - Validate API response times
- [ ] **[Arch] Cross-Cutting Type Safety**
  - Wrap `JSON.parse` with Zod
  - Remove `as unknown as` assertions

### Acceptance Criteria
- [ ] All tasks completed

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `hjps-ops/apps/pdf-generator`
- `toolkit-standards/`

### References
- [PHASE6_TASKS_LEGACY.md](../../../../archive/PHASE6_TASKS_LEGACY.md)

### Notes for AI
Atomic tasks.

---

## 3. Review (Cool-Down)

### Verification
- [ ] Manual verification

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
