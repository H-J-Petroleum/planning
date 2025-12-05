# ADR-002: HubSpot Data Modeling with Branded Types and Realistic Field Types

**Status:** Approved
**Date:** 2025-11-24
**Owner:** Architecture Team + Backend Team
**Related:** Phase 6 Type Safety Refactor, ADR-001 (Zod Runtime Validation)

## Context

**Current problems with HubSpot data types:**

1. **Over-permissive types defeat TypeScript safety:**
   ```typescript
   // Current (WRONG):
   interface DealProperties extends Record<string, unknown> { }
   interface TimesheetProperties extends Record<string, unknown> { }

   // Result: Properties are essentially untyped
   const hours = timesheet.properties.hours_worked; // Type: unknown
   ```

2. **Type/reality mismatch - HubSpot sends strings for numbers:**
   ```typescript
   // Current model (WRONG):
   hours_worked?: number;

   // HubSpot API reality:
   { "hours_worked": "40.5" } // String, not number!

   // Result: Runtime type errors
   const total = hours_worked * rate; // NaN if hours_worked is string
   ```

3. **No type safety for HubSpot IDs:**
   ```typescript
   function getTimesheet(id: string) { } // Any string accepted

   getTimesheet("random-string"); // Compiles but fails at runtime
   getTimesheet(dealId); // Wrong ID type, compiles but wrong!
   ```

4. **Inconsistent data models across codebase:**
   - `toolkit-pdf` defines `HubSpotTimesheetProperties` with realistic types
   - `toolkit-hubspot` uses `Record<string, unknown>`
   - `toolkit-testing` fixtures have detailed property shapes
   - `dataops` services redefine properties locally

**Impact:**
- No compile-time protection against misspelled property names
- Runtime `NaN` errors from string/number confusion
- ID mixups (passing contact ID where deal ID expected)
- Code duplication and drift (multiple property definitions)

## Decision

**Adopt a schema-first, reality-based approach to HubSpot data modeling:**

### 1. Branded Types for Domain IDs

Use TypeScript branded types to distinguish different HubSpot ID types:

```typescript
import { z } from 'zod';

// Branded types (compile-time only, zero runtime cost)
export const HubSpotId = z.string().min(1).brand<'HubSpotId'>();
export type HubSpotId = z.infer<typeof HubSpotId>;

export const Email = z.string().email().brand<'Email'>();
export type Email = z.infer<typeof Email>;

// Usage:
function getTimesheet(id: HubSpotId) { /* ... */ }

const timesheetId: HubSpotId = "12345"; // Error: Type 'string' is not assignable to type 'HubSpotId'
const validated = HubSpotId.parse("12345"); // OK: Runtime validation + branding
```

**Benefits:**
- Compile-time safety: Can't pass wrong ID type
- Self-documenting: `HubSpotId` is clearer than `string`
- Zero runtime cost: Brands are erased at runtime

### 2. Model External Reality - Number | String for Numeric Fields

HubSpot API sends numbers as strings. Model this explicitly:

```typescript
const NumericField = z.union([z.number(), z.string()]).optional();

export const HubSpotTimesheetSchema = z.object({
  hs_object_id: HubSpotId.optional(),
  timesheet_project_id: HubSpotId,
  timesheet_consultant_id: HubSpotId,
  timesheet_consultant_email: Email.optional(),

  // Model reality: HubSpot sends strings for numbers
  hours_worked: NumericField,
  billing_rate: NumericField,
  billing_amount: NumericField,
  timesheet_quantity: NumericField,
  timesheet_hj_price: NumericField,
  timesheet_total_price: NumericField,

  // Dates as strings (ISO 8601)
  timesheet_start_date: z.string().optional(),
  timesheet_end_date: z.string().optional(),

  // Status with fallback for future values
  timesheet_approval_status: z.string().optional(),
  approval_status: z.string().optional(),

  // Forward compatibility: allow unknown fields
}).passthrough();

export type HubSpotTimesheet = z.infer<typeof HubSpotTimesheetSchema>;
```

**Safe conversion at boundaries:**
```typescript
import { toNumber } from '@h-j-petroleum/toolkit-validation/utils';

// Helper handles string | number | null | undefined | NaN
function toNumber(value: unknown, defaultValue = 0): number {
  const n = Number(value);
  return Number.isFinite(n) ? n : defaultValue;
}

// Usage in domain logic:
calculateMetrics(timesheets: CrmRecord<HubSpotTimesheet>[]): TimesheetMetrics {
  return timesheets.reduce((metrics, ts) => {
    const props = ts.properties;
    const hours = toNumber(props.hours_worked);  // Safe: handles "40.5" or 40.5
    const rate = toNumber(props.billing_rate);    // Safe: handles "150" or 150
    const billing = toNumber(props.billing_amount ?? hours * rate);

    metrics.totalHours += hours;
    metrics.totalBilling += billing;
    return metrics;
  }, initialMetrics);
}
```

### 3. Canonical Property Interfaces in toolkit-validation

Create single source of truth for HubSpot properties:

**Package:** `@h-j-petroleum/toolkit-validation`
**File:** `src/hubspot.ts`

Define schemas for all HubSpot object types:
- `HubSpotContactSchema` → `HubSpotContact`
- `HubSpotDealSchema` → `HubSpotDeal`
- `HubSpotCompanySchema` → `HubSpotCompany`
- `HubSpotProjectSchema` → `HubSpotProject`
- `HubSpotConsultantSchema` → `HubSpotConsultant`
- `HubSpotTimesheetSchema` → `HubSpotTimesheet`
- `HubSpotApprovalSchema` → `HubSpotApproval`

**Source schemas from existing usage:**
- Reuse property definitions from `toolkit-pdf/index.d.ts` (HubSpotTimesheetProperties, HubSpotApprovalProperties)
- Align with `toolkit-testing/src/fixtures/HubSpotFixtures.ts` (realistic test data)
- Remove duplicate local definitions from `dataops` services

### 4. Generic HubSpot Object with Type Parameter

Make `HubSpotObject` generic to support typed properties:

```typescript
// Before (WEAK):
export interface HubSpotObject {
  id: string;
  properties: Record<string, unknown>; // Too permissive
  createdAt: string;
  updatedAt: string;
  archived: boolean;
}

// After (STRONG):
export interface HubSpotObject<TProps = Record<string, unknown>> {
  id: string;
  properties: TProps; // Typed!
  createdAt: string;
  updatedAt: string;
  archived: boolean;
}

// Usage:
export type HubSpotTimesheet = HubSpotObject<HubSpotTimesheetProperties>;
export type HubSpotDeal = HubSpotObject<HubSpotDealProperties>;
```

### 5. Update BaseCrmService to Accept Schema Parameter

```typescript
export class BaseCrmService<
  TProperties extends Record<string, unknown> = Record<string, unknown>
> {
  protected schema?: z.ZodType<TProperties>;

  constructor(
    protected client: HubSpotClient,
    protected objectType: string,
    schema?: z.ZodType<TProperties>
  ) {
    this.schema = schema;
  }

  async get(id: string, properties?: string[]): Promise<CrmRecord<TProperties>> {
    const record = await this.client.get(this.objectType, id, properties);

    if (this.schema) {
      try {
        const validated = this.schema.parse(record.properties);
        return { ...record, properties: validated };
      } catch (error) {
        logger.error('CRM record validation failed', {
          objectType: this.objectType,
          recordId: id,
          error,
        });
        throw error;
      }
    }

    return record as CrmRecord<TProperties>;
  }
}

// Service implementation:
export class TimesheetsService extends BaseCrmService<HubSpotTimesheet> {
  constructor(client: HubSpotClient) {
    super(client, 'timesheets', HubSpotTimesheetSchema);
  }

  // All methods now return CrmRecord<HubSpotTimesheet> with validated properties
}
```

## Consequences

### Positive

1. **Compile-time safety for HubSpot data**
   - Misspelled properties caught at compile time
   - Can't pass wrong ID types (contact ID where deal ID expected)
   - Autocomplete for HubSpot properties in IDEs

2. **Runtime correctness**
   - Explicit handling of string/number duality (no more `NaN` surprises)
   - Validation catches unexpected API changes
   - Safe numeric coercion with `toNumber` helper

3. **Code reuse and consistency**
   - Single source of truth for HubSpot properties (toolkit-validation)
   - Eliminates duplicate definitions across packages
   - Fixtures, services, and tests use same types

4. **Forward compatibility**
   - `.passthrough()` allows unknown fields from HubSpot API evolution
   - Explicit conversion at boundaries (not throughout codebase)

5. **Self-documenting code**
   - `HubSpotId` is clearer than `string`
   - `number | string` makes reality explicit
   - Schema serves as documentation

### Negative

1. **Breaking changes**
   - `Record<string, unknown>` → schema-derived types changes service interfaces
   - Consumers must update to use new types
   - Numeric fields change from `number` to `number | string` (must use `toNumber`)

2. **Verbosity**
   - Must explicitly convert at boundaries: `toNumber(props.hours_worked)`
   - Can't assume numeric fields are numbers (reality, but more code)

3. **Migration effort**
   - Update all service implementations (toolkit-hubspot, dataops)
   - Update all consumers (scripts, tests, workflows)
   - Update fixtures to match schemas

### Mitigation

1. **Breaking changes:** Gradual rollout (Week 2-4) with deprecated types during transition; documented in TYPE_SAFETY_MIGRATION_GUIDE.md
2. **Verbosity:** Helper utilities (`toNumber`, `toInteger`) reduce boilerplate; centralized in toolkit-validation
3. **Migration effort:** Phased approach (toolkits → ops → dataops); clear examples in migration guide

## Alternatives Considered

### Keep `Record<string, unknown>` + Manual Type Guards

**Pros:** No breaking changes, gradual adoption
**Cons:**
- Doesn't solve type safety problem (still untyped)
- Manual guards are verbose and error-prone
- Doesn't address string/number duality

**Decision:** Rejected; doesn't solve the root problem

### Model Numbers as Number Only (Ignore Reality)

**Pros:** Simpler types, more ergonomic
**Cons:**
- Runtime errors when HubSpot sends strings
- Requires coercion everywhere or silent `NaN` bugs
- Type system doesn't match reality

**Decision:** Rejected; types should model reality, not wishful thinking

### Separate Input/Output Types

**Pros:** Input types model API reality (`number | string`), output types are normalized (`number`)
**Cons:**
- Duplication (two sets of types)
- Complexity (when to use which type?)
- Doesn't align with Zod schema-first approach

**Decision:** Rejected; single schema with explicit conversion is clearer

### Use Template Literal Types for IDs

```typescript
type HubSpotContactId = `contact-${string}`;
type HubSpotDealId = `deal-${string}`;
```

**Pros:** More specific than branded types
**Cons:**
- HubSpot IDs don't have prefixes (just numbers)
- Doesn't add value over branded types
- Runtime validation harder

**Decision:** Rejected; branded types are sufficient and more flexible

## Implementation Timeline

**Week 2: Foundation**
- Create canonical schemas in toolkit-validation
- Define branded types (HubSpotId, Email)
- Create `toNumber` helper

**Week 3: Toolkits**
- Update `HubSpotObject<TProps>` generic in toolkit-hubspot
- Update `BaseCrmService<TProperties>` to accept schema
- Migrate service implementations (TimesheetsService, DealsService, etc.)

**Week 4: Dataops**
- Migrate dataops services to use toolkit-validation schemas
- Remove local property definitions
- Update stubs to match real API

## References

- ADR-001: Zod Runtime Validation
- Phase 6 Tasks: [docs/planning/PHASE6_TASKS.md](../planning/PHASE6_TASKS.md)
- Existing HubSpot types:
  - `adopt_hjps_toolkit/packages/toolkit-pdf/index.d.ts` (HubSpotTimesheetProperties, HubSpotApprovalProperties)
  - `adopt_hjps_toolkit/packages/toolkit-testing/src/fixtures/HubSpotFixtures.ts`
- Type Safety Analysis: Reviewer feedback in planning session
- TypeScript Index Signatures: https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures
- Zod Branded Types: https://zod.dev/?id=brand
