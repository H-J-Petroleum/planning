# Bet: Billing Integration

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
**Goal**: Automate Phase 4 (Billing) to ensure cash flow.

---

## 1. Shaping (Planning)

### Context & Problem
The current billing process involves manual downloading, renaming, logging, and Excel pivoting. This is slow and error-prone.
- **Problem:** "Debbie" manually downloads/renames files and maintains handwritten logs.
- **Gap:** No automation for the "Billing Artifact Manager" or "Digital Ledger".
- **Solution:** Implement the "Billing Artifact Manager" script and "Invoice Consolidation" logic defined in `IMPROVEMENT-PLAN.md`.

### Scope
- **In Scope**:
    - **Artifact Manager:** Script to fetch, rename, and save PDFs (`[yy/mm/dd]_[Consultant]...`).
    - **Digital Ledger:** Generate a `billing_ledger.json` (or CSV) instead of handwritten logs.
    - **Status Sync:** Auto-update "Imported Customer Invoice" and "Invoiced to QB" in HubSpot.
    - **Consolidation:** Logic to group line items by Job Type for Enverus exports.
- **Out of Scope**:
    - Direct QuickBooks API integration (CSV export is sufficient for now).
    - OCR Signature Verification (Future).

### Solution Design
1.  **Script:** `scripts/billing/artifact-manager.ts`
    - Uses `hubspot-api` to find `APPROVED` timesheets.
    - Downloads associated files to `data/billing/invoices` and `data/billing/paystubs`.
2.  **Script:** `scripts/billing/generate-ledger.ts`
    - Generates the consolidated CSV for QuickBooks/Enverus.
3.  **Automation:** Run these scripts via `npm run billing:prep`.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Phase 1: Artifact Manager**
    - [ ] Implement file download & rename logic.
    - [ ] Implement local directory organization.
- [ ] **Phase 2: Ledger Generation**
    - [ ] Implement "Consolidation Rules" (group by Job Type).
    - [ ] Generate CSVs for QuickBooks and Enverus.
- [ ] **Phase 3: Status Sync**
    - [ ] Implement bulk update back to HubSpot.

### Acceptance Criteria
- [ ] Running the script downloads all new approved PDFs with correct names.
- [ ] Script generates a CSV that matches the "Handwritten Log" requirements.
- [ ] HubSpot records are updated after run.

### Files/Repos Affected
- `hjps-ops/scripts/billing/` (New Directory)

### References
- [IMPROVEMENT-PLAN.md](../../../../hjps-ops/docs/timesheet-process/phases/04-billing/docs/IMPROVEMENT-PLAN.md)

---

## 3. Review (Cool-Down)

### Verification
- [ ] Run script against a test batch of approved timesheets.
- [ ] Verify file names and CSV output.
