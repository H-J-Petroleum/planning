# Bet: Mastra.ai Adoption

**Status**: Proposed
**Cycle**: Cycle 2
**Appetite**: 🐎 Medium
**Owner**: [Arch]
**Origin**: [Agentic Code Investigation](../../../analysis/AGENTIC_CODE_CONSOLIDATION.md)
**Goal**: [DevOS Architecture](../../../goals/devos-architecture.md)

---

## 1. Shaping (Planning)

### Context & Problem
Our current agentic infrastructure is split:
1.  **Logic**: `scripts/skills/*.ts` contains the core capabilities (e.g., `arch-dep-cruise`, `api-openapi-check`).
2.  **Protocol**: `mcp/` contains a custom implementation of the Model Context Protocol (YAML manifests, Python servers).

This separation creates maintenance overhead and relies on a custom MCP implementation rather than a robust framework. We want to move to a TypeScript-first agent structure using **Mastra.ai**.

### Scope
- **In Scope**:
    - Refactoring `scripts/skills/*.ts` to be compatible with Mastra Tools.
    - **Implementing ADR-005**: All tools must have a Manifest (Risk Level, Domain, etc.).
    - Setting up a Mastra application to serve these tools.
    - Replacing the `mcp/` directory with the Mastra implementation.
- **Out of Scope**:
    - Rewriting the core logic of the skills (unless necessary for the interface).
    - Expanding the set of skills (focus is on migration).

### Research & Options
- [x] Research conducted? [Agentic Code Consolidation Report](../../../analysis/AGENTIC_CODE_CONSOLIDATION.md)
- **Option A**: Keep custom MCP implementation (High maintenance, no framework benefits).
- **Option B**: Adopt Mastra.ai (Standardized, TS-first, robust).
- **Decision**: **Option B** - Adopt Mastra.ai.

### Solution Design
1.  **Tools Layer**: Wrap existing `scripts/skills/*.ts` functions as Mastra Tools.
    - **Governance**: Apply `ADR-005` metadata to every tool definition.
2.  **Server Layer**: Use Mastra to serve these tools via MCP.
3.  **Orchestration**: Use **Mastra Workflows** to implement the "Outer Loop" governance (Shadow vs Auto modes).
4.  **Cleanup**: Delete the legacy `mcp/` directory (YAML manifests and Python servers).

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Phase 1: Setup**
    - [ ] Initialize Mastra project in `agent/` (or similar).
    - [ ] Configure Mastra to run in the repo.
- [ ] **Phase 2: Migration & Governance**
    - [ ] Refactor `scripts/skills/arch-dep-cruise.ts` -> Mastra Tool (Risk: READ_ONLY).
    - [ ] Refactor `scripts/skills/api-openapi-check.ts` -> Mastra Tool (Risk: READ_ONLY).
    - [ ] Refactor remaining skills, applying Risk Levels.
- [ ] **Phase 3: Cleanup**
    - [ ] Verify Mastra agent works with existing clients (Claude/Cursor).
    - [ ] Delete `mcp/` directory.

### Acceptance Criteria
- [ ] All existing skills in `scripts/skills/` are available via the Mastra agent.
- [ ] The `mcp/` directory is deleted.
- [ ] Clients (Claude, Cursor) can connect to the new Mastra agent.

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below
- [x] Example or reference provided
- [x] No ambiguous requirements

### Files/Repos Affected
- `mcp/` (Delete)
- `scripts/skills/` (Refactor/Wrap)
- `agent/` (New)

### References
- [Mastra.ai Documentation](https://mastra.ai/)
- [Agentic Code Consolidation Report](../../../analysis/AGENTIC_CODE_CONSOLIDATION.md)

### Notes for AI
- Focus on wrapping the existing logic in `scripts/skills/` rather than rewriting it.
- Ensure the new Mastra agent exposes the same tool interfaces (arguments/outputs) to minimize client disruption.

---

## 3. Review (Cool-Down)

### Verification
- [ ] Automated tests passed
- [ ] Manual verification completed

### Retrospective
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
