# Spike: Assign Consultant Timesheet API Refactor

**Status**: ![Status: Ready](https://img.shields.io/badge/status%2Fready-0e8a16)
**Type**: Spike
**Owner**: Backend Team

<!-- LABEL SYSTEM METADATA -->
![Priority: Medium](https://img.shields.io/badge/priority%2Fmedium-ffc107)
![Type: Spike](https://img.shields.io/badge/type%2Fspike-0e8a16)
![Scope: Cross-Repo](https://img.shields.io/badge/scope%2Fcross--repo-0075ca)
![Effort: 3](https://img.shields.io/badge/effort%2F3-7bc96f)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

**Goal**: Move the assign-consultant CLI from “fat client against HubSpot” to a thin orchestration layer that talks to `timesheet-api`.

---

## 1. Shaping (Planning)

### Context & Problem
The `assign-consultant` CLI currently talks directly to HubSpot via `@h-j-petroleum/toolkit-hubspot` and a custom service layer. This duplicates business logic (validation, allocation, payload building) that also exists in `timesheet-api`. This leads to:
- Duplication of business rules.
- Difficulty evolving timesheet behavior.
- Limited observability of CLI-driven timesheet creation.

### Scope
- **In Scope**:
    - Refactoring CLI to use `timesheet-api` client.
    - Implementing `validate`, `create`, `submit` flows via API.
    - Removing direct HubSpot calls from CLI.
- **Out of Scope**:
    - Major changes to `timesheet-api` internal logic (unless needed for API contract).
    - UI changes in the CLI (UX should remain stable).

### Solution Design
The detailed technical plan is documented here:
[ASSIGN_CONSULTANT_TIMESHEET_API_REFACTOR_PLAN.md](file:///d:/code/hjps-ops/docs/refactoring/ASSIGN_CONSULTANT_TIMESHEET_API_REFACTOR_PLAN.md)

**Key Components:**
1.  **Timesheet API Client**: A new client in the CLI to communicate with `timesheet-api`.
2.  **API Endpoints**: Reuse/refine `validate`, `create`, `submit`, `complete` endpoints.
3.  **Feature Flags**: Use flags to incrementally roll out API-based flows.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Phase 0: Baseline + Guardrails**
  - [ ] Verify `timesheet-api` routes and behaviors.
  - [ ] Add/update tests in `packages/timesheet-api`.
- [ ] **Phase 1: Introduce Timesheet API Client**
  - [ ] Add `TimesheetApiClient` to CLI.
- [ ] **Phase 2: CLI Validation via API**
  - [ ] Replace CLI validation with `/api/timesheet/validate`.
- [ ] **Phase 3: Payload Build via API**
  - [ ] Replace payload construction with `/api/timesheet/create`.
- [ ] **Phase 4: Submission via API**
  - [ ] Replace submission with `/api/timesheet/submit`.
- [ ] **Phase 6: Cleanup**
  - [ ] Remove direct HubSpot wiring.

### Acceptance Criteria
- [ ] CLI performs all timesheet operations via `timesheet-api`.
- [ ] No regression in consultant assignment flows.
- [ ] CLI operations are observable in `timesheet-api` logs/traces.

### Files/Repos Affected
- `hjps-ops/src/cli/assign-consultant`
- `hjps-ops/packages/timesheet-api`

---

## 3. Review (Cool-Down)

### Verification
- [ ] Run CLI with `ASSIGN_CLI_USE_TIMESHEET_VALIDATE=true` and verify validation.
- [ ] Run CLI with `ASSIGN_CLI_USE_TIMESHEET_CREATE=true` and verify payload creation.
- [ ] Run CLI with `ASSIGN_CLI_USE_TIMESHEET_SUBMIT=true` and verify submission.
