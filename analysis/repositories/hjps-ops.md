# hjps-ops Adoption Requirements Analysis

**Last Updated:** 2025-11-11  
**Repository:** hjps-ops  
**Status:** Analysis Complete

---

## Executive Summary

hjps-ops is a mature operations platform with npm workspaces, mixed module systems (CommonJS + ESM), and local toolkit dependencies. This analysis identifies blockers and requirements for policy system adoption.

**Readiness:** ⚠️ Medium - Blockers exist but are addressable

---

## Current State

**✅ Ready for Adoption:**
- CI/CD already uses reusable workflows from toolkit-standards
- MCP policy configured (writes.enabled=false)
- Test coverage targets align (80%+)
- Security scanning in place
- Vitest testing framework matches plan

**⚠️ Blockers for Full Adoption:**
1. **Workspace Support** - Policy system assumes pnpm, hjps-ops uses npm workspaces
2. **File: Protocol Dependencies** - Local toolkit dependencies not validated
3. **Reusable Policy-Checks Workflow** - Not yet created in toolkit-standards
4. **Mixed Module Systems** - CommonJS + ESM not validated

---

## Specific Requirements

### 13. Monorepo/Workspace Support

**Gap:**
- **AGENTIC_IMPLEMENTATION_PLAN.md**: Assumes pnpm workspaces
- **hjps-ops Reality**: Uses npm workspaces (not pnpm)
- **Impact:** Policy system may not work correctly for npm workspace repos

**Findings:**
- hjps-ops uses `npm workspaces` with multiple packages
- CI uses `npm ci` and `npm install --workspaces`
- Policy system assumes pnpm for lockfile integrity checks
- `deps:policy-node` may not validate npm workspace dependencies correctly

**Recommendation:**
- Add `repo:workspace-check` skill to detect workspace type (npm/pnpm/yarn)
- Support npm workspaces in dependency validation
- Update `deps:policy-node` to handle npm workspace dependencies
- Add workspace-aware lockfile integrity checks

**Action Required:** Add workspace support to Phase 6

---

### 14. CI/CD Integration Patterns

**Status:** ✅ Partially Aligned

**Findings:**
- hjps-ops already uses reusable workflows from toolkit-standards:
  - `reusable-lint.yml`
  - `reusable-build.yml`
  - `reusable-secret-detection.yml`
- Uses `H-J-Petroleum/toolkit-standards/.github/workflows/` pattern
- Policy-checks workflow should follow same pattern

**Gap:**
- Policy-checks workflow not yet created in toolkit-standards
- hjps-ops would benefit from reusable policy-checks workflow
- Need to support npm workspaces in workflow

**Recommendation:**
- Create reusable policy-checks workflow in toolkit-standards
- Support both npm and pnpm workspaces
- Follow existing reusable workflow pattern
- Add to Phase 5 deliverables

**Action Required:** Add reusable policy-checks workflow to Phase 5

---

### 15. Local Toolkit Dependencies (file: protocol)

**Gap:**
- **hjps-ops Reality**: Uses `file:../hjps-toolkit/packages/*` for local toolkit packages
- **CI Challenge**: CI creates stub packages to satisfy file: dependencies
- **Policy System**: May not handle file: dependencies correctly

**Findings:**
```json
{
  "@hjps/toolkit-hubspot": "file:../hjps-toolkit/packages/toolkit-hubspot",
  "@hjps/toolkit-pdf": "file:../hjps-toolkit/packages/toolkit-pdf"
}
```

**CI Workaround:**
- CI creates stub packages in `../hjps-toolkit/packages/`
- Policy system may flag these as invalid dependencies

**Recommendation:**
- Add `deps:file-protocol-check` skill to validate file: dependencies
- Allow file: dependencies for local development
- Warn about file: dependencies in production builds
- Document file: dependency handling in adoption guide

**Action Required:** Add file: protocol support to dependency validation

---

### 16. Mixed Module Systems (CommonJS + ESM)

**Gap:**
- **hjps-ops Reality**: Mixed CommonJS (approval-api) and ESM (timesheet-api)
- **Policy System**: Assumes consistent module system
- **Vitest Config**: Complex setup for mixed modules

**Findings:**
- `packages/approval-api` uses CommonJS
- `packages/timesheet-api` uses ESM
- Vitest config handles both via aliases and pool: 'forks'
- TypeScript uses NodeNext module resolution

**Recommendation:**
- Add `arch:module-system-check` skill to detect mixed module systems
- Validate module consistency per package
- Support mixed module systems in policy checks
- Document mixed module patterns in adoption guide

**Action Required:** Add module system validation to Phase 6

---

### 17. Docker/Dev Stack Requirements

**Gap:**
- **hjps-ops Reality**: Extensive Docker Compose setup for local development
- **Policy System**: No validation for Docker/containerization standards

**Findings:**
- `docker-compose.dev-stack.yml` with multiple profiles
- Services: redis, approvals, pdf, timesheet, gateway
- Test runner service for integration tests
- Local development stack requirements

**Recommendation:**
- Add `infra:docker-check` skill to validate Dockerfile standards
- Validate docker-compose.yml structure
- Check for security best practices in Docker configs
- Document Docker standards in adoption guide

**Action Required:** Add Docker validation skills to Phase 6+ (optional)

---

### 18. Test Coverage Requirements

**Status:** ✅ Aligned

**Findings:**
- hjps-ops targets 80%+ coverage (aligned with plan)
- Uses Codecov for coverage reporting
- Coverage enabled via `COVERAGE=true` environment variable
- Vitest coverage configuration in place

**Recommendation:**
- Ensure `test:coverage` skill supports Vitest coverage reports
- Validate coverage thresholds per package in monorepo
- Support Codecov integration

**Action Required:** Verify test:coverage skill works with Vitest + Codecov

---

### 19. Security Policy Integration

**Gap:**
- **hjps-ops SECURITY.md**: Defines severity levels and response times
- **Policy System**: No validation of security policy compliance

**Findings:**
- Security severity definitions (Critical, High, Medium, Low)
- Response time requirements (24h critical, 48h high, etc.)
- Security contact information
- Vulnerability disclosure policy

**Recommendation:**
- Add `security:policy-check` skill to validate SECURITY.md presence
- Validate security contact information
- Check for security update policy
- Document security policy requirements

**Action Required:** Add security policy validation to Phase 6

---

### 20. Ecosystem Architecture Awareness

**Gap:**
- **hjps-ops ECOSYSTEM_ARCHITECTURE.md**: Documents complex multi-repo architecture
- **Policy System**: No awareness of ecosystem relationships

**Findings:**
- hjps-ops is part of 5-stage evolution plan
- Dependencies on hjps-toolkit, hjps-frontend, hubspot-dataops
- Migration phases and inter-repo dependencies
- Toolkit integration strategy

**Recommendation:**
- Add `repo:ecosystem-check` skill to validate ecosystem relationships
- Check for toolkit dependencies
- Validate inter-repo dependency versions
- Document ecosystem patterns

**Action Required:** Add ecosystem validation to Phase 6+ (optional)

---

### 21. TypeScript Configuration Patterns

**Gap:**
- **hjps-ops**: Uses `NodeNext` module resolution
- **Policy System**: May assume different TypeScript config patterns

**Findings:**
```json
{
  "module": "NodeNext",
  "moduleResolution": "NodeNext"
}
```

**Recommendation:**
- Ensure `typecheck` skill supports NodeNext resolution
- Validate TypeScript config consistency
- Support multiple module resolution strategies

**Action Required:** Verify typecheck skill works with NodeNext

---

## Pre-Adoption Checklist

- [ ] Create reusable policy-checks workflow supporting npm workspaces
- [ ] Add `repo:workspace-check` skill
- [ ] Update `deps:policy-node` for npm workspace support
- [ ] Add file: protocol dependency handling
- [ ] Verify typecheck works with NodeNext resolution
- [ ] Verify test:coverage works with Vitest + Codecov
- [ ] Document npm workspace adoption patterns

---

## Recommended Adoption Path

**Phase 1: Basic Integration (Week 1)**
- Add reusable policy-checks workflow to CI
- Run in advisory mode
- Collect baseline metrics

**Phase 2: Workspace Support (Week 2)**
- Implement workspace detection
- Update dependency validation
- Test with hjps-ops structure

**Phase 3: Full Enforcement (Week 3+)**
- Transition to warn mode
- Monitor failure rates
- Address exceptions
- Move to enforce mode when ready

---

## Risk Assessment

### Migration Risk Matrix

| Risk Factor | Likelihood | Impact | Overall Risk | Mitigation Strategy |
|-------------|------------|--------|--------------|---------------------|
| **Workspace Detection Failure** | MEDIUM | HIGH | **HIGH** | Implement `repo:workspace-check` before adoption; test with npm workspaces |
| **file: Protocol Breaking CI** | HIGH | HIGH | **CRITICAL** | Resolve via DEPENDENCY_STRATEGY.md (registry migration or CI setup) |
| **Mixed Module System Issues** | MEDIUM | MEDIUM | **MEDIUM** | Implement `arch:module-system-check`; test with both CJS and ESM packages |
| **Docker Compose Complexity** | LOW | LOW | **LOW** | Docker validation is optional; can adopt without it |
| **Team Resistance** | LOW | MEDIUM | **LOW-MEDIUM** | Start in advisory mode; 2-week training period |
| **CI/CD Performance Impact** | LOW | MEDIUM | **LOW-MEDIUM** | Monitor FAST gate execution time (<10s target) |

### Risk Details

**1. Workspace Detection Failure**
- **Likelihood:** MEDIUM - npm workspaces work differently from pnpm
- **Impact:** HIGH - Would block repository adoption entirely
- **Mitigation:**
  - Priority 1: Implement `repo:workspace-check` skill supporting npm/pnpm/yarn
  - Test extensively with hjps-ops workspace structure
  - Validate package.json workspace configuration detection
  - Ensure lockfile integrity checks work with package-lock.json

**2. file: Protocol Dependencies Breaking CI**
- **Likelihood:** HIGH - Known blocker, hardcoded directory assumptions
- **Impact:** HIGH - CI will fail without special setup
- **Mitigation:**
  - Option A (Recommended): Migrate to GitHub Packages registry (see DEPENDENCY_STRATEGY.md)
  - Option B: Create CI workaround (symlink setup in GitHub Actions)
  - Option C: Monorepo root (organizational change)
  - Timeline: Must resolve before enforce mode (Week 1-2 of Phase 6)

**3. Mixed Module System Issues**
- **Likelihood:** MEDIUM - CommonJS + ESM together is complex
- **Impact:** MEDIUM - May cause validation errors or false positives
- **Mitigation:**
  - Implement `arch:module-system-check` skill
  - Support NodeNext module resolution
  - Validate both CJS (approval-api) and ESM (timesheet-api) packages
  - Document mixed-module patterns as acceptable

**4. Docker Compose Complexity**
- **Likelihood:** LOW - Docker validation is optional
- **Impact:** LOW - Does not block adoption
- **Mitigation:**
  - Defer `infra:docker-check` skill to Phase 7+
  - Focus on core adoption first
  - Add Docker validation after stable adoption

**5. Team Resistance to Policy System**
- **Likelihood:** LOW - Team already uses reusable workflows
- **Impact:** MEDIUM - Could slow adoption if team pushes back
- **Mitigation:**
  - Start in advisory mode (no blocking, data collection only)
  - 2-week training period with clear documentation
  - Address concerns proactively
  - Show metrics demonstrating value

**6. CI/CD Performance Impact**
- **Likelihood:** LOW - Policy system is optimized for speed
- **Impact:** MEDIUM - Could slow down PR process
- **Mitigation:**
  - Monitor FAST gate execution time (target: <10 seconds)
  - Optimize slow skills or move to FULL gate
  - Parallel execution where possible

### Rollback Triggers

If any of the following occur, consider rolling back or pausing adoption:

1. **PR Time-to-Green increases by >50%** - Policy checks are too slow
2. **Failure rate >25% after 2 weeks in warn mode** - Too many false positives
3. **Team velocity decreases by >30%** - Policy system is blocking productivity
4. **Critical production issue caused by policy enforcement** - Safety issue
5. **Unable to resolve file: protocol issue within 2 weeks** - Technical blocker

### Rollback Procedure

1. **Immediate:** Switch enforcement mode to `advisory`
2. **Within 24 hours:** Document what went wrong
3. **Within 1 week:** Create action plan to address issues
4. **Re-attempt:** Only after blockers are resolved

### Risk Score Summary

**Overall Migration Risk: MEDIUM-HIGH**
- **Rationale:** file: protocol dependencies and workspace support are significant blockers, but both are addressable with clear mitigation paths
- **Confidence:** MEDIUM - Similar patterns seen in other repositories
- **Recommendation:** Address HIGH priority blockers (workspace support, file: protocol) before adoption

---

## Success Criteria

- [ ] Policy-checks workflow runs successfully in CI
- [ ] All workspace packages validated correctly
- [ ] File: protocol dependencies handled gracefully
- [ ] Mixed module systems supported
- [ ] Test coverage validated per package
- [ ] TypeScript validation works with NodeNext
- [ ] PR failure rate < 5% after adoption

---

## Skills Needed

1. `repo:workspace-check` - npm/pnpm workspace detection
2. `deps:file-protocol-check` - file: dependency validation
3. `arch:module-system-check` - CommonJS/ESM validation
4. `infra:docker-check` - Docker standards (optional)
5. `security:policy-check` - SECURITY.md validation

---

**See Also:**
- Complete Review: `../../IMPLEMENTATION_PLAN_STANDARDS_REVIEW.md`
- Summary: `../SUMMARY.md`
- Phase 6 Priorities: `../recommendations/phase-6-priorities.md`

