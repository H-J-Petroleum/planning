# Bet: Frontend Toolkit Readiness

**Status**: Shaping 🟡
**Cycle**: 2
**Appetite**: 🐘 Large (2-3 days)
**Owner**: [FE]
**Goal**: [Phase 6 Adoption](../../../goals/phase-6-adoption.md)

---

## 1. Shaping (Planning)

### Context & Problem
The frontend toolkit needs to be production-ready to support the GUI migration. It currently lacks a coherent structure and published packages.

### Scope
- **In Scope**: `frontend-toolkit` monorepo structure, core components, initial publish.
- **Out of Scope**: Migrating existing apps (Cycle 3).

### Research & Options
- **Decision**: Use `pnpm` workspace with `changesets` (standard).

### Solution Design
Organize as a monorepo with `packages/ui`, `packages/hooks`, `packages/utils`.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Monorepo Structure**
  - [ ] Organize packages correctly
- [ ] **Design System**
  - [ ] Implement core components (Button, Input, Card)
- [ ] **Initial Publish**
  - [ ] Publish to GitHub Packages

### Acceptance Criteria
- [ ] Packages published to `@h-j-petroleum/ui`, etc.
- [ ] Components have Storybook entries (optional but recommended)

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `frontend-toolkit/`

### References
- [Design System Specs](link-to-figma-if-exists)

### Notes for AI
Follow the `hjps-toolkit` structure.

---

## 3. Review (Cool-Down)

### Verification
- [ ] `npm install @h-j-petroleum/ui` works

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
