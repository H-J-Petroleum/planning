# Bet: Critical Skills Implementation

**Status**: Shaping 🟡
**Cycle**: 2
**Appetite**: 🐘 Large (2-3 days)
**Owner**: [Arch]
**Goal**: [Phase 6 Adoption](../../../goals/phase-6-adoption.md)

---

## 1. Shaping (Planning)

### Context & Problem
We need to implement the remaining critical skills to enforce architectural boundaries and performance budgets. Without these, we risk regression during the migration.

### Scope
- **In Scope**: `arch:layer-check`, `perf:bundle-budget-check`, `api:openapi-check`.
- **Out of Scope**: Fixing existing violations (that is a separate cleanup task).

### Research & Options
- **Option A (Manual Review)**: Too slow and error-prone.
- **Option B (Automated Skills)**: Add to `standards-cli`.
- **Decision**: **Option B** for continuous enforcement.

### Solution Design
Implement new skills in `standards-cli` using the existing skill architecture.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **`arch:layer-check`**
  - [ ] Enforce Core/IO separation
  - [ ] Block imports from `logic/` to `adapters/`
- [ ] **`perf:bundle-budget-check`**
  - [ ] Enforce bundle size limits
- [ ] **`api:openapi-check`**
  - [ ] Validate API implementations against OpenAPI specs

### Acceptance Criteria
- [ ] `standards-cli check` fails on layer violations
- [ ] `standards-cli check` fails on bundle size exceeded
- [ ] `standards-cli check` fails on OpenAPI mismatch

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `toolkit-standards/packages/standards-cli/src/core/skills/`

### References
- [DevOS Architecture Canvas](../../../../DEVOS_ARCHITECTURE_CANVAS.md)

### Notes for AI
Reuse existing AST traversal logic in `standards-cli`.

---

## 3. Review (Cool-Down)

### Verification
- [ ] Create a violation -> Check fails
- [ ] Fix violation -> Check passes

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
