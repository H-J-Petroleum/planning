# Bet: GUI Portals Development

**Status**: Shaping 🟡
**Cycle**: 2
**Appetite**: 🐘 Large (2-3 days)
**Owner**: [FE]
**Goal**: [Phase 6 Adoption](../../../goals/phase-6-adoption.md)

---

## 1. Shaping (Planning)

### Context & Problem
We need to build the `approval-portal` and `timesheet-portal` using the new frontend toolkit to validate its usability and provide the required UI.

### Scope
- **In Scope**: Scaffold apps, implement core flows.
- **Out of Scope**: Full feature parity with legacy apps (MVP only).

### Research & Options
- **Decision**: Use Next.js 15 (App Router) as per standard.

### Solution Design
Two separate Next.js apps consuming `@h-j-petroleum/ui`.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Approval Portal**
  - [ ] Scaffold app
  - [ ] Implement core flows (List, Approve/Reject)
- [ ] **Timesheet Portal**
  - [ ] Scaffold app
  - [ ] Implement core flows (Submit, View)

### Acceptance Criteria
- [ ] Apps build and run locally
- [ ] Apps consume toolkit packages
- [ ] Core flows functional

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `frontend-toolkit/apps/approval-portal`
- `frontend-toolkit/apps/timesheet-portal`

### References
- [Next.js Docs](https://nextjs.org/docs)

### Notes for AI
Use `create-next-app` with TypeScript and Tailwind.

---

## 3. Review (Cool-Down)

### Verification
- [ ] Manual walkthrough of flows

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
