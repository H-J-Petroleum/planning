# Bet: DataOps Type Safety

**Status**: ![Status: Proposed](https://img.shields.io/badge/status%2Fproposed-d4c5f9)
**Cycle**: 3
**Owner**: Ops

<!-- LABEL SYSTEM METADATA -->
![Priority: Medium](https://img.shields.io/badge/priority%2Fmedium-ffc107)
![Type: Cleanup](https://img.shields.io/badge/type%2Fcleanup-d4c5f9)
![Scope: Single Repo](https://img.shields.io/badge/scope%2Fsingle--repo-0075ca)
![Effort: 5](https://img.shields.io/badge/effort%2F5-7bc96f)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

**Goal**: [Phase 6 Adoption](../../../goals/phase-6-adoption.md)

---

## 1. Shaping (Planning)

### Context & Problem
Eliminate `any` types in DataOps and enforce Zod validation. DataOps is critical and currently lacks type safety.

### Scope
- **In Scope**: `dataops` repository.

### Research & Options
- **Decision**: Zod for runtime validation.
- **Testing Strategy**: Adopt 4-layer strategy from [ADR-003](../../../../adr/ADR-003-VALIDATION-TESTING-STRATEGY.md).

### Solution Design
Replace `any` with Zod schemas and validate all IO.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Replace `Record<string, any>`**
  - [ ] Use Zod schemas
- [ ] **Validate PostgreSQL Results**
  - [ ] Enforce schema validation on DB results
- [ ] **Testing (ADR-003)**
  - [ ] Layer 1: Type-level tests
  - [ ] Layer 2: Unit tests
  - [ ] Layer 3: Integration tests

### Acceptance Criteria
- [ ] Zero `any` types
- [ ] All DB queries validated
- [ ] 4-layer tests passing

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `dataops/`

### References
- [Zod Docs](https://zod.dev)
- [ADR-003](../../../../adr/ADR-003-VALIDATION-TESTING-STRATEGY.md)

### Notes for AI
Be strict about types.

---

## 3. Review (Cool-Down)

### Verification
- [ ] `tsc` passes with `noImplicitAny`

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
