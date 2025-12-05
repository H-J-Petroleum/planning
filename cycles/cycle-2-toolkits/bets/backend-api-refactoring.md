# Bet: Backend API Refactoring

**Status**: ![Status: In Progress](https://img.shields.io/badge/status%2Fin--progress-1d76db)
**Cycle**: 2
**Owner**: Ops

<!-- LABEL SYSTEM METADATA -->
![Priority: High](https://img.shields.io/badge/priority%2Fhigh-ff9800)
![Type: Cleanup](https://img.shields.io/badge/type%2Fcleanup-d4c5f9)
![Scope: Multi Repo](https://img.shields.io/badge/scope%2Fmulti--repo-7057ff)
![Effort: 5](https://img.shields.io/badge/effort%2F5-7bc96f)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

**Origin**: [User Request]
**Goal**: Refactor `timesheet-api` and `approval-api` to consume the shared `timesheet-logic` library and expose it to the Frontend.

---

## 1. Shaping (Planning)

### Context & Problem
We are extracting logic into a shared library (`timesheet-logic`) and building a Frontend (`hjps-frontend`). We need the API layer to connect them.
- **Problem:** Existing APIs (`timesheet-api`) contain mixed logic and direct DB calls.
- **Gap:** They need to be refactored to use the new shared logic library.
- **Solution:** "Thin Controller" pattern. The API controllers should just call the shared `timesheet-logic` functions.

### Scope
- **In Scope**:
    - **Refactor:** `timesheet-api` to use `packages/timesheet-logic`.
    - **Refactor:** `approval-api` to use `packages/timesheet-logic`.
    - **OpenAPI:** Generate OpenAPI 3.0 schema for `hjps-superdesign` consumption.
- **Out of Scope**:
    - Rewrite of legacy endpoints not used in MVP.

### Solution Design
1.  **Shared Lib:** Ensure `timesheet-logic` exports clean service functions.
2.  **Controllers:** Update controllers to import from `timesheet-logic`.
3.  **Validation:** Use Zod schemas from `timesheet-logic` for request validation.
4.  **OpenAPI:** Use `tsoa` or similar to generate `openapi.json` from controllers.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Phase 1: Dependency Injection**
    - [ ] Link `timesheet-logic` package.
- [ ] **Phase 2: Controller Refactor**
    - [ ] Strip business logic out of `timesheet-api`.
    - [ ] Replace with calls to shared library.
- [ ] **Phase 3: Validation**
    - [ ] Ensure Zod schemas match between API and Logic.
- [ ] **Phase 4: OpenAPI Generation**
    - [ ] Configure OpenAPI generator.
    - [ ] Verify schema output.

### Acceptance Criteria
- [ ] APIs delegate all complex logic to the shared library.
- [ ] Frontend can perform all Phase 2/3 actions via these APIs.
- [ ] OpenAPI schema is generated and valid.

### Files/Repos Affected
- `hjps-ops/packages/timesheet-api/`
- `hjps-ops/packages/approval-api/`

### References
- [Timesheet Agent Bet](../bets/timesheet-agent.md) (Source of Logic)

---

## 3. Review (Cool-Down)

### Verification
- [ ] API integration tests.
