# Bet: Toolkit Refactoring

**Status**: Shaping 🟡
**Cycle**: 2
**Appetite**: 🐎 Medium (Half Day)
**Owner**: [BE]
**Goal**: [Phase 6 Adoption](../../../goals/phase-6-adoption.md)

---

## 1. Shaping (Planning)

### Context & Problem
Refactor `toolkit-hubspot` to strictly follow the Core/IO separation and use Zod schemas. This is the pilot for the larger service refactoring.

### Scope
- **In Scope**: `toolkit-hubspot`.
- **Out of Scope**: Other toolkits.

### Research & Options
- **Decision**: Use Zod for all IO validation.
- **Testing Strategy**: Adopt 4-layer strategy from [ADR-003](../../../../adr/ADR-003-VALIDATION-TESTING-STRATEGY.md).

### Solution Design
Separate logic into `src/logic` and IO into `src/adapters`.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Core/IO Separation**
  - [ ] Move logic to `src/logic`
  - [ ] Move IO to `src/adapters`
- [ ] **Zod Schemas**
  - [ ] Harden all inputs/outputs with Zod
- [ ] **Testing (ADR-003)**
  - [ ] Layer 1: Type-level tests (`expect-type`)
  - [ ] Layer 2: Unit tests for schemas
  - [ ] Layer 3: Integration tests with fixtures
  - [ ] Layer 4: Property-based tests (optional)

### Acceptance Criteria
- [ ] No `any` types
- [ ] All IO validated
- [ ] 4-layer tests passing

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `hjps-toolkit/packages/toolkit-hubspot`

### References
- [Zod Docs](https://zod.dev)
- [ADR-003](../../../../adr/ADR-003-VALIDATION-TESTING-STRATEGY.md)

### Notes for AI
Ensure backward compatibility where possible.

---

## 3. Review (Cool-Down)

### Verification
- [ ] `pnpm test` passes

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
