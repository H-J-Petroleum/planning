# Bet: pnpm Standardization

**Status**: ![Status: Ready](https://img.shields.io/badge/status%2Fready-0e8a16)
**Cycle**: Backlog
**Owner**: Arch

<!-- LABEL SYSTEM METADATA -->
![Priority: Low](https://img.shields.io/badge/priority%2Flow-8bc34a)
![Type: Cleanup](https://img.shields.io/badge/type%2Fcleanup-d4c5f9)
![Scope: Multi Repo](https://img.shields.io/badge/scope%2Fmulti--repo-7057ff)
![Effort: 5](https://img.shields.io/badge/effort%2F5-7bc96f)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

---

## 1. Shaping (Planning)

### Context & Problem
We need to enforce pnpm 10.x and Node 24 across all repositories to ensure consistent builds and leverage latest performance features.

### Scope
- **In Scope**: All 11 repositories.
- **Out of Scope**: Legacy archived repos.

### Research & Options
- **Option A (Docs)**: Just write it in README. Will be ignored.
- **Option B (Code)**: Enforce via `standards-cli` skill.
- **Decision**: **Option B** for automated enforcement.

### Solution Design
We will implement a `repo:workspace-check` skill in `standards-cli` to enforce these versions via `package.json` engines and constraints.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Add `repo:workspace-check` skill**
  - [ ] File: `packages/standards-cli/src/core/skills/repo-workspace-check.ts`
  - [ ] Enforce `packageManager: pnpm@10.x`
  - [ ] Enforce `engines.node: >=24`
- [ ] **Update Constraints**
  - [ ] File: `policy/node/constraints.json`
  - [ ] Add bans: `winston`, `jest`, `joi`
  - [ ] Add floors: `React >=19`, `Next >=15.5.x`
  - [ ] Add Zod: `^4.x`
- [ ] **Reusable Workflow**
  - [ ] Ensure `pnpm/action-setup` is used

### Acceptance Criteria
- [ ] `standards-cli check` fails if pnpm version is < 10
- [ ] `standards-cli check` fails if node version is < 24
- [ ] All repos pass the check after update

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `toolkit-standards/packages/standards-cli/src/core/skills/repo-workspace-check.ts`
- `toolkit-standards/policy/node/constraints.json`

### References
- [pnpm 10 Release Notes](https://pnpm.io/blog)
- [Node 24 Release Notes](https://nodejs.org/en/blog)

### Notes for AI
Use `semver` package for version comparison.

---

## 3. Review (Cool-Down)

### Verification
- [ ] Run `standards-cli check` on a repo with old pnpm -> Fails
- [ ] Run `standards-cli check` on a repo with pnpm 10 -> Passes

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
