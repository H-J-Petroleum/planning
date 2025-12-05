# Bet: Timesheet Agent

**Status**: ![Status: In Progress](https://img.shields.io/badge/status%2Fin--progress-1d76db)
**Cycle**: 2
**Owner**: AI

<!-- LABEL SYSTEM METADATA -->
![Priority: Medium](https://img.shields.io/badge/priority%2Fmedium-ffc107)
![Type: Feature](https://img.shields.io/badge/type%2Ffeature-0e8a16)
![Scope: Single Repo](https://img.shields.io/badge/scope%2Fsingle--repo-0075ca)
![Effort: 5](https://img.shields.io/badge/effort%2F5-7bc96f)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

**Goal**: [Phase 6 Adoption](../../../goals/phase-6-adoption.md)

---

## 1. Shaping (Planning)

### Context & Problem
The current timesheet system is broken and requires manual intervention. We have a defined process (`docs/timesheet-process`) but lack the implementation.
- **Problem:** Manual validation of timesheets is time-consuming and error-prone.
- **Role:** The Agent acts as an **Auditor/Advisor**, NOT an Approver.
- **Goal:** Prevent mistakes/fraud by flagging anomalies *before* the Customer sees them.
- **Solution:** Implement Phase 2/3 logic and an Agent that "audits" every submission.

### Scope
- **In Scope**:
    - **Phase 1 Logic:** Implement `01_foundation` logic (Project Config, Approver Assignment) required for validation.
    - **Phase 2 Logic:** Implement `02_timesheet_creation/backend/core-logic.js` (Time Entry, Validation).
    - **Phase 3 Logic:** Implement `03_approval/backend/core-logic.js` (Approval Workflow, Rejection).
    - **Agent (Auditor):** Implement `03_approval/agents/timesheet-agent`.
        - **Consultant Check:** "Pre-flight" validation (e.g., "You forgot to attach a receipt").
        - **Fraud/Error Check:** Anomaly detection (e.g., "Hours > 20% over average").
    - **Feedback Loop:** Agent writes validation flags to `Timesheet.validation_flags` (DB), which are displayed in the UI.
    - **Migration:** Extract logic from `assign-consultant` CLI into these shared modules.
- **Out of Scope**:
    - Phase 4 (Billing) - External system.

### Solution Design
1.  **Structure:** Follow `D:\code\hjps-ops\docs\timesheet-process` structure strictly.
2.  **Logic:**
    - Extract `pricing` and `validation` logic from CLI.
    - Place in `phases/02-timesheet-creation/backend`.
3.  **Agent:**
    - The Agent is just a consumer of this logic.
    - It uses `phases/03-approval/backend` to validate submissions.
    - **Feedback:** Agent updates the Timesheet record with a `validation_status` and `flags` array.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Phase 1: Logic Extraction (The "Brain")**
    - [ ] Create `packages/timesheet-logic` (Shared Library).
    - [ ] Port CLI logic for "Scope" and "Pricing" to this library.
    - [ ] Map to `02_timesheet_creation/backend`.
- [ ] **Phase 2: Agent Implementation (The "Auditor")**
    - [ ] Implement `TimesheetAgent` using Mastra.
    - [ ] Connect it to `timesheet-logic` for validation rules.
    - [ ] Connect it to RAG for historical context.
    - [ ] Implement `write_validation_flags` tool.
- [ ] **Phase 3: Integration**
    - [ ] Expose logic via API (`timesheet-api`).
    - [ ] Verify Agent can "audit" a timesheet using this logic.

### Acceptance Criteria
- [ ] Logic for Time Entry and Approval exists in shared modules.
- [ ] Agent can validate a timesheet using this shared logic.
- [ ] Agent feedback (flags) is visible in the Database (and thus UI).
- [ ] Structure matches `docs/timesheet-process`.

### Files/Repos Affected
- `hjps-ops/packages/timesheet-logic/` (New Shared Lib)
- `hjps-ops/docs/timesheet-process/` (Implementation)

### References
- [PROCESS-FLOW-COMPLETE.md](../../../../hjps-ops/docs/timesheet-process/PROCESS-FLOW-COMPLETE.md)
- [CLI Master Plan](../../../../hjps-ops/docs/feat-scope-service-enhancements/HJ_MANAGEMENT_CLI_MASTER_PLAN.md)

---

## 3. Review (Cool-Down)

### Verification
- [ ] Automated tests for shared logic.
- [ ] Agent verification run.
