# Bet: Speed Gates

**Status**: ![Status: Ready](https://img.shields.io/badge/status%2Fready-0e8a16)
**Cycle**: Backlog
**Owner**: Ops

<!-- LABEL SYSTEM METADATA -->
![Priority: Medium](https://img.shields.io/badge/priority%2Fmedium-ffc107)
![Type: Automation](https://img.shields.io/badge/type%2Fautomation-1d76db)
![Scope: Multi Repo](https://img.shields.io/badge/scope%2Fmulti--repo-7057ff)
![Effort: 5](https://img.shields.io/badge/effort%2F5-7bc96f)
![AI: Ready](https://img.shields.io/badge/ai%2Fready-7057ff)

---

## 1. Shaping (Planning)

### Context & Problem
CI gates must be fast (< 2 min) to keep developer velocity high. Currently, they run on all files. We need change-aware checks.

### Scope
- **In Scope**: `standards-cli` verification logic.
- **Out of Scope**: Changing the CI provider (GitHub Actions).

### Research & Options
- **Option A (Turborepo)**: Good but complex setup.
- **Option B (Git Diff)**: Simple script in `standards-cli`.
- **Decision**: **Option B** for immediate impact with low complexity.

### Solution Design
We will update `scripts/policy-verify.ts` to detect changed files via git diff and scope `prettier` and `eslint` checks to those files only.

---

## 2. Execution (Building)

### Implementation Plan
- [ ] **Change-Aware FAST Scope**
  - [ ] Update `scripts/policy-verify.ts`
  - [ ] Implement git diff discovery
  - [ ] Apply to `prettier` and `eslint`
- [ ] **Move `build:validation`**
  - [ ] Move to FULL gate by default
- [ ] **Cache Security Scans**
  - [ ] Cache by lockfile hash

### Acceptance Criteria
- [ ] FAST gate runs in < 30s for single file change
- [ ] FULL gate runs in < 5m
- [ ] Security scans use cache

### AI-Readiness Checklist
- [x] Problem clearly defined
- [x] Acceptance criteria are testable
- [x] Related files/repos linked below

### Files/Repos Affected
- `toolkit-standards/scripts/policy-verify.ts`

### References
- [GitHub Actions Caching](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)

### Notes for AI
Use `git diff --name-only origin/main...HEAD` to find changed files.

---

## 3. Review (Cool-Down)

### Verification
- [ ] Change 1 file -> Check runs in < 30s
- [ ] Change no files -> Check runs in < 10s

### Retrospective
<!-- Fill this in during Cool-Down -->
- **Successes**: ...
- **Challenges**: ...
- **Learnings**: ...
