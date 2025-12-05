# Bet: Small Fixes Batch

**Status**: ![Status: Ready](https://img.shields.io/badge/status%2Fready-0e8a16)
**Cycle**: Backlog
**Owner**: Mix

<!-- LABEL SYSTEM METADATA -->
![Priority: Low](https://img.shields.io/badge/priority%2Flow-8bc34a)
![Type: Cleanup](https://img.shields.io/badge/type%2Fcleanup-d4c5f9)
![Scope: Multi Repo](https://img.shields.io/badge/scope%2Fmulti--repo-7057ff)
![Effort: 1](https://img.shields.io/badge/effort%2F1-c2e0c6)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

---

## 1. Shaping (Planning)

### Context & Problem
A collection of small, critical fixes identified in the integration analysis that don't warrant separate bets.

### Scope
- **In Scope**: Specific tasks listed below.
- **Out of Scope**: Any major refactoring.

### Research & Options
- **Decision**: Batch these into a single "Small Bet" to avoid overhead.

### Solution Design
Execute as a checklist of independent tasks.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **[Ops] Fix DataOps Entry Points** (R5-1)
  - Create `adopt_hubspot_dataops/src/index.ts`
- [ ] **[FE] Add Missing OTEL Core Dependency** (DI-3)
  - Add `@opentelemetry/core` to `adopt_hjps_frontend`
- [ ] **[Arch] Fix standards-cli Missing Prettier** (DI-1)
  - Add prettier peerDependency
- [ ] **[Arch] Add `deps:file-protocol-check`**
- [ ] **[Arch] Add `deps:publish-check`**
- [ ] **[Ops] Remove Unused Dependencies**
  - Remove lodash, date-fns from `adopt_hubspot_dataops`

### Acceptance Criteria
- [ ] All tasks completed
- [ ] No regressions in build

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `dataops/package.json`
- `frontend-toolkit/package.json`
- `toolkit-standards/package.json`

### References
- [Integration Analysis](../../../analysis/INTEGRATION_ANALYSIS.md)

### Notes for AI
Treat each item as an atomic task.

---

## 3. Review (Cool-Down)

### Verification
- [ ] All listed items verified manually

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
