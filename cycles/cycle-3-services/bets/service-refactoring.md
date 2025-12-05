# Bet: Service Refactoring

**Status**: ![Status: Proposed](https://img.shields.io/badge/status%2Fproposed-d4c5f9)
**Cycle**: 3
**Owner**: Arch

<!-- LABEL SYSTEM METADATA -->
![Priority: Medium](https://img.shields.io/badge/priority%2Fmedium-ffc107)
![Type: Cleanup](https://img.shields.io/badge/type%2Fcleanup-d4c5f9)
![Scope: Multi Repo](https://img.shields.io/badge/scope%2Fmulti--repo-7057ff)
![Effort: 8](https://img.shields.io/badge/effort%2F8-51af32)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

**Goal**: [Phase 6 Adoption](../../../goals/phase-6-adoption.md)

---

## 1. Shaping (Planning)

### Context & Problem
Refactor core services to follow the Core/IO separation pattern. This ensures testability and maintainability.

### Scope
- **In Scope**: `approval-api`, `timesheet-api`, `dataops` workflows.
- **Out of Scope**: Legacy services not being migrated.

### Research & Options
- **Decision**: Standard Hexagonal Architecture.

### Solution Design
Move business logic to `core/` and infrastructure to `adapters/`.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **`approval-api`**
  - [ ] Refactor Core/IO
- [ ] **`timesheet-api`**
  - [ ] Refactor Core/IO
- [ ] **`dataops` Workflows**
  - [ ] Refactor Core/IO
- [ ] **Mark Extraction Candidates**
  - [ ] Identify reusable patterns (e.g., `toolkit-workflow`)
  - [ ] Add TODOs for future extraction

### Acceptance Criteria
- [ ] No IO in Core
- [ ] No Logic in Adapters
- [ ] Tests pass
- [ ] Extraction candidates marked

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `hjps-ops/apps/approval-api`
- `hjps-ops/apps/timesheet-api`
- `dataops/workflows`

### References
- [DevOS Architecture Canvas](../../../../DEVOS_ARCHITECTURE_CANVAS.md)
- [EXECUTION_PLAN.md](../../../EXECUTION_PLAN.md)

### Notes for AI
Follow the pattern established in `toolkit-hubspot`.

---

## 3. Review (Cool-Down)

### Verification
- [ ] `standards-cli check` passes `arch:layer-check`

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
