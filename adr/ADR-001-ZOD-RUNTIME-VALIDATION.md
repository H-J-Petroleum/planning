# ADR-001: Zod for Runtime Validation

**Status:** Approved
**Date:** 2025-11-24
**Owner:** Architecture Team
**Related:** Phase 6 Type Safety Refactor

## Context

The codebase has grown to include extensive TypeScript types for domain models (HubSpot objects, API responses, database schemas), but **lacks runtime validation** of external data. This creates a critical gap:

**Current problems:**
1. **No validation at boundaries:** External data from HubSpot API, PostgreSQL queries, JSON parsing, and YAML config is trusted without verification
2. **Type assertions without safety:** Patterns like `response.data as Type` and `JSON.parse(content) as Config` bypass TypeScript's type safety
3. **Silent failures:** Invalid external data causes runtime errors deep in business logic instead of failing fast at boundaries
4. **Security vulnerabilities:** Missing input sanitization enables command injection (environment validator spawning PowerShell)

**Business impact:**
- HubSpot API changes break production without compile-time detection
- Malformed data propagates into business logic calculations (timesheet metrics, approval workflows)
- Security risk: unsanitized inputs in shell commands
- No observability: validation failures not tracked or logged

**Architecture context:**
- Phase 6 establishes hexagonal architecture (ports/adapters pattern)
- Observability infrastructure ready (TimescaleDB + vector embeddings)
- Phase 7 self-healing requires structured validation failure tracking

## Decision

**Adopt Zod as the standard runtime validation library** for all external data boundaries.

**Strategy:**
1. **Validation at IO boundaries only** - Don't validate internal domain logic; trust typed data once validated
2. **Schema-first approach** - Define Zod schemas as single source of truth, derive TypeScript types via `z.infer<>`
3. **Observable validation** - Log all validation failures to TimescaleDB with vector embeddings
4. **Forward compatibility** - All schemas use `.passthrough()` to allow unknown fields from API evolution

**Implementation pattern:**
```typescript
import { z } from 'zod';
import { logger } from '@h-j-petroleum/toolkit-logger';

const HubSpotTimesheetSchema = z.object({
  timesheet_project_id: HubSpotId,
  timesheet_consultant_id: HubSpotId,
  hours_worked: z.union([z.number(), z.string()]).optional(),
  // ... other fields
}).passthrough(); // Allow unknown HubSpot fields

export type HubSpotTimesheet = z.infer<typeof HubSpotTimesheetSchema>;

// Use at IO boundary (HubSpotClient adapter)
async get(id: string): Promise<HubSpotObject<HubSpotTimesheet>> {
  const record = await this.client.get('timesheets', id);

  try {
    const validated = HubSpotTimesheetSchema.parse(record.properties);
    return { ...record, properties: validated };
  } catch (error) {
    logger.error('HubSpot validation failed', {
      objectType: 'timesheets',
      recordId: id,
      error: error.errors,
    });
    throw error;
  }
}
```

**Centralized validation infrastructure:**
- New package: `@h-j-petroleum/toolkit-validation`
- Location: `adopt_hjps_toolkit/packages/toolkit-validation`
- Exports: Zod schemas, branded types, validation helpers, type guards

## Consequences

### Positive

1. **Type safety with runtime guarantees** - TypeScript types backed by actual runtime validation
2. **Fail fast** - Invalid data rejected at boundaries before corrupting business logic
3. **Observable failures** - All validation errors logged to TimescaleDB with embeddings
4. **Phase 7 enabler** - Validation failures feed into self-healing loop:
   - Detect: Validation failures tracked in observability
   - Retrieve: Vector similarity search finds similar past failures
   - Propose: Agent suggests schema updates or data fixes
   - Apply: Automated PR with schema adjustments
5. **Better DX** - Single source of truth (Zod schema) instead of maintaining parallel types and validators
6. **OpenAPI generation** - Can generate API contracts from Zod schemas (supports `api:openapi-check` skill)

### Negative

1. **Runtime overhead** - Zod parsing adds ~1-5ms per validation (acceptable for API boundaries)
2. **Bundle size** - Zod adds ~15KB gzipped (minimal impact)
3. **Learning curve** - Team needs to learn Zod API (mitigated by excellent docs and TypeScript integration)
4. **Breaking changes** - HubSpot service types change from `Record<string, any>` to schema-derived types

### Mitigation strategies

1. **Performance:** Validate only at boundaries, not in hot paths or business logic
2. **Bundle size:** Tree-shaking eliminates unused schemas; acceptable for backend services
3. **Learning curve:** Provide concrete examples in ADR-002 and TYPE_SAFETY_MIGRATION_GUIDE.md
4. **Breaking changes:** Gradual rollout (Week 2-4) with deprecated types during transition

## Alternatives Considered

### io-ts
**Pros:** Mature, functional programming approach, good TypeScript integration
**Cons:**
- More verbose syntax (`t.type({ ... })` vs `z.object({ ... })`)
- Less active development (Zod has stronger community momentum)
- No built-in OpenAPI generation
- Steeper learning curve (fp-ts ecosystem)

**Decision:** Rejected in favor of Zod's better DX and ecosystem

### Manual type guards
**Pros:** No dependencies, full control, minimal overhead
**Cons:**
- Verbose and error-prone (must manually write guards for every type)
- Hard to maintain (schema changes require updating multiple guards)
- No schema introspection or OpenAPI generation
- Duplication with TypeScript types (two sources of truth)

**Decision:** Rejected due to maintenance burden; use for simple cases only

### joi / yup
**Pros:** Mature validation libraries, wide adoption
**Cons:**
- Not TypeScript-first (types must be manually maintained separately)
- Banned in `policy/node/constraints.json` (joi explicitly)
- yup has weaker type inference than Zod

**Decision:** Rejected; Zod is the modern TypeScript-first standard

### JSON Schema
**Pros:** Language-agnostic, good for OpenAPI contracts
**Cons:**
- Not TypeScript-native (requires codegen or manual type mapping)
- Verbose JSON syntax vs Zod's fluent TypeScript API
- No type inference in TypeScript

**Decision:** Rejected for primary validation; may use for OpenAPI generation from Zod schemas

## Implementation Timeline

**Week 2: Foundation**
- Create toolkit-validation package
- Define HubSpot schemas (Contact, Deal, Company, Project, Consultant, Timesheet, Approval)
- Add validation helpers (toNumber, sanitizeForShell)

**Week 3: Toolkits**
- Integrate into toolkit-hubspot (BaseCrmService validation)
- Wrap JSON.parse() in dashboard scripts
- Validate environment config + sanitize shell inputs

**Week 4: Dataops + Standards**
- Integrate into dataops services (TimesheetService, ApprovalService, ProjectService)
- Implement `validation:schema-check` skill
- Create migration guide

## References

- Phase 6 Tasks: [docs/planning/PHASE6_TASKS.md](../planning/PHASE6_TASKS.md)
- DevOS Architecture: [docs/DEVOS_ARCHITECTURE_CANVAS.md](../DEVOS_ARCHITECTURE_CANVAS.md) (§3-4: Hexagonal architecture)
- Observability Plan: [docs/planning/observability/IMPLEMENTATION_PLAN.md](../planning/observability/IMPLEMENTATION_PLAN.md)
- Zod Documentation: https://zod.dev
- Related ADRs: ADR-002 (HubSpot Data Modeling), ADR-003 (Validation Testing Strategy)
