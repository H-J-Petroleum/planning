# Bet: Repo Identity

**Status**: ![Status: Ready](https://img.shields.io/badge/status%2Fready-0e8a16)
**Cycle**: Backlog
**Owner**: Arch

<!-- LABEL SYSTEM METADATA -->
![Priority: Low](https://img.shields.io/badge/priority%2Flow-8bc34a)
![Type: Cleanup](https://img.shields.io/badge/type%2Fcleanup-d4c5f9)
![Scope: Multi Repo](https://img.shields.io/badge/scope%2Fmulti--repo-7057ff)
![Effort: 1](https://img.shields.io/badge/effort%2F1-c2e0c6)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

---

## 1. Shaping (Planning)

### Context & Problem
Repository names are inconsistent (`hjps-frontend` vs `hubspot-dataops`). Ambiguous naming confuses developers and agents about the purpose of a repository (Toolkit vs Platform).

### Scope
- **In Scope**: Renaming `hjps-frontend` and `hubspot-dataops`.
- **Out of Scope**: Renaming `hjps-ops` or `hjps-toolkit` (already correct).

### Research & Options
- **Option A (Prefixes)**: Rename everything to `hjps-*`. Hard to read.
- **Option B (Functional)**: Rename to `frontend-toolkit`, `dataops`. Clearer intent.
- **Decision**: **Option B** for clarity and agent context.

### Solution Design
We will rename repositories to match their architectural role and document the naming convention.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Document Classification**
  - [ ] Create `docs/REPOSITORY_ARCHITECTURE.md`
  - [ ] Define Toolkits vs Platforms
- [ ] **Rename Repos**
  - [ ] `hjps-frontend` → `frontend-toolkit`
  - [ ] `hubspot-dataops` → `dataops`
  - [ ] Update `package.json` names
  - [ ] Update GitHub descriptions

### Acceptance Criteria
- [ ] `git clone .../frontend-toolkit` works
- [ ] `git clone .../dataops` works
- [ ] `package.json` names match repo names

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `hjps-frontend/package.json`
- `hubspot-dataops/package.json`

### References
- [GitHub Renaming Docs](https://docs.github.com/en/repositories/creating-and-managing-repositories/renaming-a-repository)

### Notes for AI
When renaming, ensure you update the local directory names as well if you are performing the operation.

---

## 3. Review (Cool-Down)

### Verification
- [ ] `git remote -v` shows new URLs
- [ ] CI/CD pipelines updated to new names

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
