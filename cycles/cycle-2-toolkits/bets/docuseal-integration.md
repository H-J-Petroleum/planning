# Bet: DocuSeal Integration

**Status**: ![Status: In Progress](https://img.shields.io/badge/status%2Fin--progress-1d76db)
**Cycle**: 2
**Owner**: Ops

<!-- LABEL SYSTEM METADATA -->
![Priority: Medium](https://img.shields.io/badge/priority%2Fmedium-ffc107)
![Type: Feature](https://img.shields.io/badge/type%2Ffeature-0e8a16)
![Scope: Single Repo](https://img.shields.io/badge/scope%2Fsingle--repo-0075ca)
![Effort: 5](https://img.shields.io/badge/effort%2F5-7bc96f)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

**Origin**: [User Request]
**Goal**: Integrate DocuSeal for legally binding timesheet approvals.

---

## 1. Shaping (Planning)

### Context & Problem
The "Timesheet Relief" product requires not just approval, but **signed** approval.
- **Problem:** We have an `approval-api` but it lacks the signature layer.
- **Gap:** Phase 3 (Approval) requires DocuSeal integration as per `PROCESS-FLOW-COMPLETE.md`.
- **Solution:** Implement a `DocuSealService` in `approval-api` that handles the signing workflow.

### Scope
- **In Scope**:
    - **Service:** Create `DocuSealService` in `packages/approval-api`.
    - **Workflow:** Trigger signature request on "Submit for Approval".
    - **Callback:** Handle DocuSeal webhooks to update status to "Signed".
    - **Storage:** Store the signed PDF URL in the Timesheet record.
- **Out of Scope**:
    - Complex multi-party signing chains (start with 1:1 Consultant -> Approver).

### Solution Design
1.  **Trigger:** When `Timesheet` status -> `SUBMITTED`, call DocuSeal API to create a template instance.
2.  **Signing:** Email sent to Approver (or Consultant first, then Approver).
3.  **Completion:** Webhook hits `approval-api`, updates status to `APPROVED` (or `SIGNED`).

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Phase 1: SDK Integration**
    - [ ] Install DocuSeal Node.js SDK (or use fetch).
    - [ ] Configure API Keys in `approval-api`.
- [ ] **Phase 2: Service Implementation**
    - [ ] Implement `createSignatureRequest(timesheetId, signerEmail)`.
    - [ ] Implement `handleWebhook(payload)`.
- [ ] **Phase 3: Integration**
    - [ ] Hook into the `submit` workflow in `approval-api`.

### Acceptance Criteria
- [ ] Submitting a timesheet triggers a DocuSeal email.
- [ ] Signing the document updates the Timesheet status in the DB.
- [ ] Signed PDF link is available via API.

### Files/Repos Affected
- `hjps-ops/packages/approval-api/`

### References
- [PROCESS-FLOW-COMPLETE.md](../../../../hjps-ops/docs/timesheet-process/PROCESS-FLOW-COMPLETE.md)

---

## 3. Review (Cool-Down)

### Verification
- [ ] Manual test: Submit timesheet -> Sign email -> Check DB status.
