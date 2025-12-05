# Bet: Standards Dogfooding

**Status**: Shaping 🟡
**Cycle**: 2
**Appetite**: 🐎 Medium (Half Day)
**Owner**: [Arch]
**Goal**: [Phase 6 Adoption](../../../goals/phase-6-adoption.md)

---

## 1. Shaping (Planning)

### Context & Problem
We must validate the standards on the standards repo itself. If we can't follow our own rules, we can't expect others to.

### Scope
- **In Scope**: `toolkit-standards` repo.
- **Out of Scope**: Other repos.

### Research & Options
- **Decision**: Enforce strict compliance.

### Solution Design
Run `standards-cli` against `toolkit-standards` and fix all violations.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Run Skills**
  - [ ] Run all skills on `toolkit-standards`
- [ ] **Refactor CLI**
  - [ ] Ensure `standards-cli` follows Core/IO separation

### Acceptance Criteria
- [ ] `toolkit-standards` passes all applicable skills
- [ ] `standards-cli` architecture is compliant

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `toolkit-standards/`

### References
- [DevOS Architecture Canvas](../../../../DEVOS_ARCHITECTURE_CANVAS.md)

### Notes for AI
Be strict.

---

## 3. Review (Cool-Down)

### Verification
- [ ] `standards-cli check` passes on itself

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
