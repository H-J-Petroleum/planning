# Bet: Small Fixes Batch

**Status**: Shaping 🟡
**Cycle**: 2
**Appetite**: 🐇 Small (1-2 hours)
**Owner**: [Mix]
**Goal**: [Phase 6 Adoption](../../../goals/phase-6-adoption.md)

---

## 1. Shaping (Planning)

### Context & Problem
Small fixes for Cycle 2, including missing observability components and performance checks.

### Scope
- **In Scope**: Listed tasks.

### Research & Options
- **Decision**: Batch execution.

### Solution Design
Checklist execution.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **[BE] Add `obs:otel-check`**
- [ ] **[Arch] Create Dependency Version Manifest**
- [ ] **[Arch] Implement `TimescaleExporter`** in `toolkit-metrics`
  - Enable metrics in `approval-api`
- [ ] **[FE] Add `perf:lighthouse-check`**
  - Validate performance/accessibility

### Acceptance Criteria
- [ ] All tasks completed

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `toolkit-standards/`
- `toolkit-metrics/`

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
