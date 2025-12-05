# hjps-frontend Adoption Requirements Analysis

**Last Updated:** 2025-11-11  
**Repository:** hjps-frontend  
**Status:** Analysis Complete

---

## Executive Summary

hjps-frontend is a Next.js 15.5.4 frontend application with React 19, Tailwind CSS 4, and comprehensive observability. It uses npm (not pnpm), has bundle budget enforcement, and integrates with Superdesign for AI-powered feature generation.

**Readiness:** ⚠️ Medium - Frontend-specific requirements needed

---

## Current State

**✅ Ready for Adoption:**
- CI/CD already uses reusable workflows from toolkit-standards
- MCP policy configured (writes.enabled=false)
- Vitest testing framework matches plan
- TypeScript configuration aligned
- Security scanning in place
- OpenTelemetry observability implemented

**⚠️ Blockers for Full Adoption:**
1. **Bundle Budget Enforcement** - Has script but not integrated into policy system
2. **OpenAPI Client Generation** - Validation not in policy system
3. **Next.js Specific Patterns** - App Router, middleware patterns not validated
4. **Frontend Performance Standards** - Web Vitals not enforced
5. **Environment Variable Validation** - Custom env/ directory not validated
6. **Superdesign Integration** - AI-powered generation patterns not validated

---

## Specific Requirements

### 37. Bundle Budget Enforcement

**Gap:**
- **hjps-frontend Reality**: Has `scripts/enforce-bundle-budget.js` with 25KB gzipped budget
- **Policy System**: No bundle budget validation skill
- **PERF_STANDARDS.md**: Defines bundle budgets (150KB initial, 50KB per chunk)

**Findings:**
- Bundle budget: 25KB gzipped per route (more strict than PERF_STANDARDS.md)
- Uses `nextjs-bundle-analysis` for analysis
- Budget configured in `package.json`: `nextBundleAnalysis.budget: 25600` (25KB)
- Has `analyze:bundle` script
- Bundle diff workflow exists (`.github/workflows/bundle-diff.yml`)

**Recommendation:**
- Add `perf:bundle-budget-check` skill to validate bundle sizes
- Integrate with existing `enforce-bundle-budget.js` script
- Support Next.js bundle analysis format
- Validate against PERF_STANDARDS.md budgets (150KB initial, 50KB per chunk)
- Document frontend-specific bundle budgets

**Action Required:** Add bundle budget validation to Phase 6

---

### 38. OpenAPI Client Generation

**Gap:**
- **hjps-frontend Reality**: Generates OpenAPI client from `openapi.json`
- **Policy System**: No validation for OpenAPI client generation

**Findings:**
- Uses `openapi-typescript` for type generation
- Generates types to `src/gen/schema/`
- Has `codegen` script: `node scripts/codegen.mjs`
- CI checks for clean OpenAPI client generation diff
- OpenAPI config: `openapi.config.ts`

**Recommendation:**
- Add `api:openapi-check` skill to validate OpenAPI client generation
- Check for clean generation (no uncommitted changes)
- Validate OpenAPI schema validity
- Check for type generation errors
- Document OpenAPI integration patterns

**Action Required:** Add OpenAPI validation to Phase 6

---

### 39. Next.js App Router Patterns

**Gap:**
- **hjps-frontend Reality**: Uses Next.js 15 App Router with specific patterns
- **Policy System**: No validation for Next.js-specific patterns

**Findings:**
- App Router structure (`src/app/`)
- Route handlers (`src/app/api/`)
- Middleware (`middleware.ts`)
- Server components and client components
- Typed routes enabled
- Standalone output mode

**Recommendation:**
- Add `frontend:nextjs-check` skill to validate Next.js patterns
- Check for proper App Router structure
- Validate middleware configuration
- Check for proper component patterns
- Validate route handler patterns

**Action Required:** Add Next.js validation to Phase 6+ (optional)

---

### 40. Frontend Performance Standards

**Gap:**
- **hjps-frontend Reality**: Implements performance optimizations
- **Policy System**: No validation for frontend performance standards

**Findings:**
- Bundle budget enforcement (25KB per route)
- Smart query caching (5 min default)
- Optimized bundle size
- Next.js 15.5.4 (latest)
- Performance optimizations in place

**PERF_STANDARDS.md Requirements:**
- TTFB p95 < 200ms (static), < 350ms (SSR)
- Web Vitals: INP p75 < 200ms, LCP p75 < 2.5s, CLS p75 < 0.1
- Bundle budgets: 150KB gzipped initial route, 50KB per chunk
- Lighthouse perf ≥ 0.90

**Recommendation:**
- Add `perf:lighthouse-check` skill for Web Vitals validation
- Add `perf:bundle-budget-check` skill (see item 37)
- Validate performance optimizations
- Check for Lighthouse CI integration

**Action Required:** Add frontend performance skills to Phase 6

---

### 41. Environment Variable Management

**Gap:**
- **hjps-frontend Reality**: Custom `env/` directory with registration system
- **Policy System**: No validation for environment variable patterns

**Findings:**
- Environment variables in `env/` directory (not `.env` files)
- Registration system: `env/register.mjs`, `env/register.js`
- Manifest: `env/manifest.json`
- Loader: `env/loadEnv.js`
- Validation script: `scripts/validate-env.mjs`
- SOPS encryption for secrets

**Recommendation:**
- Add `env:validation-check` skill to validate environment setup
- Check for `env/manifest.json` presence
- Validate environment variable documentation
- Check for SOPS encryption for secrets
- Document environment patterns

**Action Required:** Add environment validation to Phase 6+ (optional)

---

### 42. Superdesign Integration

**Gap:**
- **hjps-frontend Reality**: Integrates Superdesign for AI-powered feature generation
- **Policy System**: No validation for AI generation patterns

**Findings:**
- Superdesign fork for full-stack feature generation
- Generates TypeScript React components
- Generates React Query hooks
- Generates Next.js API routes
- Generates Vitest tests
- Integration plan documented

**Recommendation:**
- Add `ai:generation-check` skill to validate AI-generated code patterns
- Check for proper code generation standards
- Validate generated code quality
- Check for proper test generation
- Document AI generation patterns

**Action Required:** Add AI generation validation to Phase 6+ (optional)

---

### 43. Playwright E2E Testing

**Gap:**
- **hjps-frontend Reality**: Uses Playwright for E2E testing
- **Policy System**: No validation for E2E testing patterns

**Findings:**
- Playwright configuration: `playwright.config.ts`
- E2E tests in `tests/e2e/`
- Smoke test: `tests/e2e/smoke.spec.ts`
- Pre-playwright script: `preplaywright` installs dependencies

**Recommendation:**
- Add `test:e2e-check` skill to validate E2E testing setup
- Check for Playwright configuration
- Validate E2E test structure
- Check for proper test coverage

**Action Required:** Add E2E testing validation to Phase 6+ (optional)

---

### 44. TypeScript Bundler Resolution

**Gap:**
- **hjps-frontend Reality**: Uses `"moduleResolution": "Bundler"` (Next.js specific)
- **Policy System**: May assume different TypeScript config patterns

**Findings:**
```json
{
  "module": "ESNext",
  "moduleResolution": "Bundler"
}
```

**Recommendation:**
- Ensure `typecheck` skill supports Bundler resolution
- Validate TypeScript config for Next.js
- Support Next.js-specific TypeScript patterns

**Action Required:** Verify typecheck skill works with Bundler resolution

---

### 45. Docker Configuration

**Status:** ✅ Partially Aligned

**Findings:**
- Dockerfile present
- docker-compose.yml present
- Standalone output mode for Docker
- Environment variable injection via SOPS

**Recommendation:**
- Ensure `infra:docker-check` skill (if implemented) validates Next.js Docker patterns
- Check for proper standalone build configuration
- Validate Docker multi-stage builds

**Action Required:** Verify Docker validation covers Next.js patterns

---

### 46. Observability Integration

**Status:** ✅ Well Aligned

**Findings:**
- OpenTelemetry Web SDK (browser)
- OpenTelemetry Node SDK (server)
- Full stack instrumentation
- Health check endpoint: `/api/health`
- Structured logging (Pino)

**Recommendation:**
- Ensure `obs:otel-check` skill (if implemented) validates OTel setup
- Check for proper trace correlation
- Validate health check endpoints

**Action Required:** Verify observability validation works with Next.js

---

## Pre-Adoption Checklist

- [ ] Add `perf:bundle-budget-check` skill for bundle size validation
- [ ] Add `api:openapi-check` skill for OpenAPI client generation
- [ ] Add `perf:lighthouse-check` skill for Web Vitals
- [ ] Verify typecheck works with Bundler resolution
- [ ] Support Next.js App Router patterns
- [ ] Document frontend-specific adoption patterns

---

## Recommended Adoption Path

**Phase 1: Basic Integration (Week 1)**
- Add reusable policy-checks workflow to CI
- Run in advisory mode
- Collect baseline metrics

**Phase 2: Frontend-Specific Support (Week 2)**
- Implement bundle budget validation
- Add OpenAPI client generation validation
- Test with hjps-frontend structure

**Phase 3: Full Enforcement (Week 3+)**
- Transition to warn mode
- Monitor failure rates
- Address exceptions
- Move to enforce mode when ready

---

## Success Criteria

- [ ] Policy-checks workflow runs successfully in CI
- [ ] Bundle budgets validated correctly
- [ ] OpenAPI client generation validated
- [ ] Web Vitals validation working
- [ ] TypeScript validation works with Bundler resolution
- [ ] Next.js patterns supported
- [ ] PR failure rate < 5% after adoption

---

## Skills Needed

1. `perf:bundle-budget-check` - Bundle size validation
2. `api:openapi-check` - OpenAPI client generation validation
3. `perf:lighthouse-check` - Web Vitals validation
4. `frontend:nextjs-check` - Next.js patterns (optional)
5. `env:validation-check` - Environment variable validation (optional)
6. `test:e2e-check` - E2E testing validation (optional)

---

**See Also:**
- Complete Review: `../../IMPLEMENTATION_PLAN_STANDARDS_REVIEW.md` (sections 37-46)
- Summary: `../SUMMARY.md`
- Phase 6 Priorities: `../recommendations/phase-6-priorities.md`

