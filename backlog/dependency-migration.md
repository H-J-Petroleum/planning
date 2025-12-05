# Bet: Dependency Migration

**Status**: ![Status: Ready](https://img.shields.io/badge/status%2Fready-0e8a16)
**Cycle**: Backlog
**Owner**: Arch

<!-- LABEL SYSTEM METADATA -->
![Priority: Medium](https://img.shields.io/badge/priority%2Fmedium-ffc107)
![Type: Cleanup](https://img.shields.io/badge/type%2Fcleanup-d4c5f9)
![Scope: Multi Repo](https://img.shields.io/badge/scope%2Fmulti--repo-7057ff)
![Effort: 8](https://img.shields.io/badge/effort%2F8-51af32)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

---

## 1. Shaping (Planning)

### Context & Problem
We must eliminate `file:` dependencies to allow for proper versioning and CI/CD. `dataops` currently relies on local paths.

### Scope
- **In Scope**: `dataops`, `hjps-ops`.
- **Out of Scope**: Toolkits themselves (they are the producers).

### Research & Options
- **Option A (Submodules)**: Painful DX.
- **Option B (Registry)**: Publish to GitHub Packages.
- **Decision**: **Option B** is the industry standard.

### Solution Design
We will replace `file:` paths in `package.json` with published `@h-j-petroleum/*` package versions.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Migrate `dataops`**
  - [ ] Replace `file:../vector-toolkit` with `@vector-toolkit/*`
  - [ ] Replace `file:../hjps-toolkit` with `@h-j-petroleum/*`
- [ ] **Verify All Repos**
  - [ ] Run `repo:dep-source` skill

### Acceptance Criteria
- [ ] No `file:` dependencies in `package.json`
- [ ] `pnpm install` succeeds
- [ ] Application builds successfully

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `dataops/package.json`
- `hjps-ops/package.json`

### References
- [pnpm Workspace Protocol](https://pnpm.io/workspaces)

### Notes for AI
Ensure you use the correct scope `@h-j-petroleum` for all packages.

---

## 3. Review (Cool-Down)

### Verification
- [ ] `pnpm install` works without local sibling folders present
- [ ] CI passes in isolated environment

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
