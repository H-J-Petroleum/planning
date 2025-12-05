# Bet: Toolkit Publishing Infrastructure

**Status**: ![Status: In Progress](https://img.shields.io/badge/status%2Fin--progress-1d76db)
**Cycle**: 2
**Owner**: Arch

<!-- LABEL SYSTEM METADATA -->
![Priority: Critical](https://img.shields.io/badge/priority%2Fcritical-d93f0b)
![Type: Automation](https://img.shields.io/badge/type%2Fautomation-1d76db)
![Scope: Multi Repo](https://img.shields.io/badge/scope%2Fmulti--repo-7057ff)
![Effort: 8](https://img.shields.io/badge/effort%2F8-51af32)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

**Goal**: [Phase 6 Adoption](../../../goals/phase-6-adoption.md)

---

## 1. Shaping (Planning)

### Context & Problem
Currently, `vector-toolkit` and `frontend-toolkit` are consumed via `file:` paths or not at all (in the case of frontend-toolkit). This breaks CI/CD in isolated environments and prevents versioning. We need to publish them to GitHub Packages.

### Scope
- **In Scope**: `vector-toolkit`, `frontend-toolkit` publishing setup.
- **Out of Scope**: Migrating consumers (that is a separate bet: Dependency Migration).

### Research & Options
- **Option A (Monorepo Tools)**: Use Lerna/Nx. Overkill for our current scale.
- **Option B (Changesets)**: Use `@changesets/cli`. Already standard in `hjps-toolkit`.
- **Decision**: **Option B** to maintain consistency with existing tooling.

### Solution Design
We will use Changesets for versioning and GitHub Actions for publishing, mirroring the setup in `hjps-toolkit`.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **vector-toolkit**
  - [ ] Add `.changeset/config.json`
  - [ ] Create `.github/workflows/release.yml`
  - [ ] Configure `.npmrc`
  - [ ] Publish initial `0.1.0` versions
- [ ] **frontend-toolkit**
  - [ ] Add changesets configuration
  - [ ] Create release workflow
  - [ ] Configure `.npmrc`
  - [ ] Publish initial `0.1.0`

### Acceptance Criteria
- [ ] `vector-toolkit` packages are visible in GitHub Package Registry
- [ ] `frontend-toolkit` packages are visible in GitHub Package Registry
- [ ] Release workflow runs successfully on main branch push

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below
- [x] Example or reference provided

### Files/Repos Affected
- `vector-toolkit/.github/workflows/release.yml`
- `frontend-toolkit/.github/workflows/release.yml`
- `hjps-toolkit/.github/workflows/release.yml` (Reference)

### References
- [Changesets Documentation](https://github.com/changesets/changesets)
- [GitHub Packages Registry](https://docs.github.com/en/packages)

### Notes for AI
Use the `hjps-toolkit` configuration as the gold standard. Copy the workflow file verbatim if possible, adjusting only the package scope.

---

## 3. Review (Cool-Down)

### Verification
- [ ] Packages are visible in GitHub Package Registry
- [ ] `npm install @h-j-petroleum/vector-toolkit` works in a fresh repo

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
