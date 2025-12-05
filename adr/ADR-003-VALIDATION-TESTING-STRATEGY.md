# ADR-003: Multi-Layer Validation Testing Strategy

**Status:** Approved
**Date:** 2025-11-24
**Owner:** Architecture Team
**Related:** Phase 6 Type Safety Refactor, ADR-001 (Zod Runtime Validation), ADR-002 (HubSpot Data Modeling)

## Context

**Current testing gaps for type safety and validation:**

1. **No tests for type system correctness**
   - Branded types could silently break (no compile-time verification in tests)
   - Schema inference not validated (types might diverge from schemas)
   - Generic type constraints not tested

2. **No validation logic tests**
   - Zod schemas untested (don't know if they accept/reject correct inputs)
   - Helpers like `toNumber`, `sanitizeForShell` untested
   - Edge cases (NaN, null, undefined, empty strings) not covered

3. **No end-to-end validation tests**
   - HubSpot API → Zod parsing → TypeScript types flow untested
   - Don't know if fixtures match schemas
   - Service methods might bypass validation

4. **No property-based testing**
   - Critical validation logic (numeric coercion, sanitization) tested with few manual examples
   - Edge cases discoverable via property-based testing missed

**Business impact:**
- Schema changes could break production without test coverage
- Validation helpers might have bugs for edge cases
- No confidence in type safety improvements
- Can't catch type system regressions

**Architecture context:**
- Phase 6 requires ≥80% test coverage for all packages
- Phase 7 self-healing requires validation failure observability
- TimescaleDB test_failures table ready for structured failure tracking

## Decision

**Adopt a multi-layer testing strategy** with four complementary testing approaches:

### Layer 1: Type-Level Tests (Compile-Time Guarantees)

**Tool:** `expect-type` (built into Vitest)
**Purpose:** Verify type system correctness at compile time

**File pattern:** `*.test-d.ts` (type-only tests, no runtime)

**Test cases:**
```typescript
import { expectTypeOf } from 'expect-type';
import {
  HubSpotId,
  HubSpotTimesheetSchema,
  type HubSpotTimesheet
} from '@h-j-petroleum/toolkit-validation/hubspot';

// Test 1: Branded types don't accept plain strings
expectTypeOf<HubSpotId>().not.toMatchTypeOf<string>();

// Test 2: Schema inference produces correct type
type InferredTimesheet = z.infer<typeof HubSpotTimesheetSchema>;
expectTypeOf<InferredTimesheet>().toEqualTypeOf<HubSpotTimesheet>();

// Test 3: Numeric fields are modeled as union
expectTypeOf<HubSpotTimesheet['hours_worked']>().toEqualTypeOf<number | string | undefined>();

// Test 4: Branded IDs not interchangeable
type ContactId = string & { __brand: 'HubSpotContactId' };
type DealId = string & { __brand: 'HubSpotDealId' };
expectTypeOf<ContactId>().not.toMatchTypeOf<DealId>();
```

**Benefits:**
- Zero runtime cost (runs at compile time)
- Catches type system regressions
- Documents type expectations

### Layer 2: Unit Tests for Validation Logic

**Tool:** Vitest
**Purpose:** Verify Zod schemas and helpers work correctly

**File pattern:** `*.test.ts` (standard unit tests)

**Test cases:**

```typescript
import { describe, it, expect } from 'vitest';
import { HubSpotTimesheetSchema, toNumber } from '@h-j-petroleum/toolkit-validation';

describe('HubSpotTimesheetSchema', () => {
  // Success path: Valid data
  it('validates correct timesheet data', () => {
    const validTimesheet = {
      timesheet_project_id: 'proj_123',
      timesheet_consultant_id: 'cons_456',
      hours_worked: '40.5',
      billing_rate: 150,
    };

    expect(() => HubSpotTimesheetSchema.parse(validTimesheet)).not.toThrow();
  });

  // Success path: Handles numeric strings
  it('accepts numeric fields as strings or numbers', () => {
    const result = HubSpotTimesheetSchema.parse({
      timesheet_project_id: 'proj_123',
      timesheet_consultant_id: 'cons_456',
      hours_worked: '40.5',     // String
      billing_rate: 150,        // Number
    });

    expect(result.hours_worked).toBe('40.5');
    expect(result.billing_rate).toBe(150);
  });

  // Success path: Allows extra fields (passthrough)
  it('allows unknown fields for forward compatibility', () => {
    const result = HubSpotTimesheetSchema.parse({
      timesheet_project_id: 'proj_123',
      timesheet_consultant_id: 'cons_456',
      future_field_added_by_hubspot: 'some_value',
    });

    expect(result).toHaveProperty('future_field_added_by_hubspot');
  });

  // Failure path: Missing required fields
  it('rejects missing required fields', () => {
    expect(() => HubSpotTimesheetSchema.parse({
      hours_worked: 40, // Missing required IDs
    })).toThrow(z.ZodError);
  });

  // Edge case: Null/undefined handling
  it('handles null/undefined for optional fields', () => {
    const result = HubSpotTimesheetSchema.parse({
      timesheet_project_id: 'proj_123',
      timesheet_consultant_id: 'cons_456',
      hours_worked: null,       // null
      billing_rate: undefined,  // undefined
    });

    expect(result.hours_worked).toBeNull();
    expect(result.billing_rate).toBeUndefined();
  });
});

describe('toNumber', () => {
  it('converts string numbers to numbers', () => {
    expect(toNumber('42')).toBe(42);
    expect(toNumber('3.14')).toBe(3.14);
    expect(toNumber('  150.5  ')).toBe(150.5); // Whitespace
  });

  it('passes through numbers unchanged', () => {
    expect(toNumber(42)).toBe(42);
    expect(toNumber(3.14)).toBe(3.14);
  });

  it('handles null/undefined with default', () => {
    expect(toNumber(null, 10)).toBe(10);
    expect(toNumber(undefined, 20)).toBe(20);
  });

  it('handles NaN/invalid with default', () => {
    expect(toNumber('invalid', 0)).toBe(0);
    expect(toNumber({}, 0)).toBe(0);
    expect(toNumber([], 0)).toBe(0);
  });

  it('handles Infinity with default', () => {
    expect(toNumber(Infinity, 0)).toBe(0);
    expect(toNumber(-Infinity, 0)).toBe(0);
  });
});

describe('sanitizeForShell', () => {
  it('allows safe characters', () => {
    expect(sanitizeForShell('my-secret_name.123')).toBe('my-secret_name.123');
  });

  it('rejects command injection attempts', () => {
    expect(() => sanitizeForShell('secret; rm -rf /')).toThrow();
    expect(() => sanitizeForShell('secret | cat /etc/passwd')).toThrow();
    expect(() => sanitizeForShell('secret$(whoami)')).toThrow();
    expect(() => sanitizeForShell('../../../etc/passwd')).toThrow();
  });
});
```

### Layer 3: Integration Tests with Fixtures

**Tool:** Vitest
**Purpose:** Verify end-to-end flow from API → Zod → TypeScript types

**File pattern:** `tests/integration/*.test.ts`

**Test cases:**

```typescript
import { describe, it, expect } from 'vitest';
import { HubSpotFixtures } from '@h-j-petroleum/toolkit-testing';
import { HubSpotTimesheetSchema } from '@h-j-petroleum/toolkit-validation/hubspot';
import { TimesheetsService } from '../../src/services/TimesheetsService';

describe('Timesheet API → Zod → TypeScript', () => {
  it('parses fixture timesheet data', () => {
    const fixtureTimesheet = HubSpotFixtures.timesheet();

    // Validate fixture matches our schema
    const parsed = HubSpotTimesheetSchema.parse(fixtureTimesheet.properties);

    expect(parsed.timesheet_project_id).toBeDefined();
    expect(parsed.timesheet_consultant_id).toBeDefined();
  });

  it('handles real HubSpot API response shape', async () => {
    const mockClient = createMockHubSpotClient({
      get: async () => HubSpotFixtures.timesheet(),
    });

    const service = new TimesheetsService(mockClient);
    const timesheet = await service.get('123');

    // Should be fully typed and validated
    expect(timesheet.properties.timesheet_project_id).toBeDefined();

    // Type safety: properties are HubSpotTimesheet, not Record<string, unknown>
    const hours = toNumber(timesheet.properties.hours_worked);
    expect(typeof hours).toBe('number');
  });

  it('calculates metrics with safe numeric coercion', async () => {
    const mockClient = createMockHubSpotClient({
      search: async () => [
        HubSpotFixtures.timesheet({ hours_worked: '40.5', billing_rate: '150' }),
        HubSpotFixtures.timesheet({ hours_worked: 35, billing_rate: 175 }),
      ],
    });

    const service = new TimesheetsService(mockClient);
    const timesheets = await service.search({});
    const metrics = service.calculateMetrics(timesheets);

    expect(metrics.totalHours).toBe(75.5); // 40.5 + 35
    expect(metrics.totalBilling).toBeGreaterThan(0);
  });
});
```

### Layer 4: Property-Based Tests (Edge Case Discovery)

**Tool:** `fast-check`
**Purpose:** Discover edge cases via randomized testing

**File pattern:** `*.test.ts` (alongside unit tests)

**Test cases:**

```typescript
import fc from 'fast-check';
import { toNumber, toInteger } from '@h-j-petroleum/toolkit-validation/utils';

describe('toNumber (property-based)', () => {
  it('always returns a finite number', () => {
    fc.assert(
      fc.property(fc.anything(), (value) => {
        const result = toNumber(value, 0);
        expect(Number.isFinite(result)).toBe(true);
      })
    );
  });

  it('numeric strings convert to their numeric value', () => {
    fc.assert(
      fc.property(fc.double(), (num) => {
        const str = String(num);
        const result = toNumber(str);

        if (Number.isFinite(num)) {
          expect(result).toBe(num);
        } else {
          expect(result).toBe(0); // Default for Infinity/NaN
        }
      })
    );
  });

  it('default value is used for invalid inputs', () => {
    fc.assert(
      fc.property(
        fc.oneof(fc.constant(null), fc.constant(undefined), fc.constant(NaN), fc.object()),
        fc.integer(),
        (invalidValue, defaultValue) => {
          const result = toNumber(invalidValue, defaultValue);
          expect(result).toBe(defaultValue);
        }
      )
    );
  });
});

describe('sanitizeForShell (property-based)', () => {
  it('safe characters always pass through', () => {
    const safeChar = fc.constantFrom(
      ...'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_-.'.split('')
    );

    fc.assert(
      fc.property(fc.array(safeChar, { minLength: 1, maxLength: 50 }), (chars) => {
        const input = chars.join('');
        expect(() => sanitizeForShell(input)).not.toThrow();
        expect(sanitizeForShell(input)).toBe(input);
      })
    );
  });

  it('unsafe characters always throw', () => {
    const unsafeChar = fc.constantFrom(';', '|', '&', '$', '`', '(', ')', '<', '>', '\n', '\r');

    fc.assert(
      fc.property(fc.string(), unsafeChar, (prefix, unsafeChar) => {
        const input = prefix + unsafeChar;
        expect(() => sanitizeForShell(input)).toThrow();
      })
    );
  });
});
```

## Consequences

### Positive

1. **Comprehensive coverage**
   - Type-level tests catch compile-time regressions
   - Unit tests verify schemas and helpers work correctly
   - Integration tests prove end-to-end flow
   - Property-based tests discover edge cases

2. **Confidence in validation**
   - 100% coverage requirement enforced
   - Known edge cases handled
   - Regression protection for schema changes

3. **Fast feedback**
   - Type-level tests run at compile time (instant)
   - Unit tests are fast (<1ms per test)
   - Property-based tests run 100+ random examples quickly

4. **Observable test failures**
   - Integration with Phase 6 test_failures table (TimescaleDB)
   - Failures logged with vector embeddings
   - Enables Phase 7 self-healing (detect → retrieve → propose → apply)

5. **Documentation value**
   - Tests serve as usage examples
   - Property-based tests document invariants
   - Integration tests show realistic workflows

### Negative

1. **More code to maintain**
   - Four test layers instead of one
   - Property-based tests harder to debug

2. **Learning curve**
   - Team needs to learn `expect-type`, `fast-check`
   - Property-based testing requires different mindset

3. **Slower test runs**
   - Property-based tests run 100+ examples (but still fast, ~10-50ms per test)
   - Integration tests may need setup/teardown

### Mitigation

1. **Maintenance:** Tests are co-located with code; schema changes force test updates
2. **Learning curve:** Provide concrete examples in this ADR and migration guide
3. **Performance:** Run property-based tests only in CI for critical paths; unit tests in watch mode

## Testing Requirements by Layer

### Required for All Zod Schemas

**Type-level:**
- ✅ Schema inference produces correct type
- ✅ Branded types properly branded (not assignable to base type)
- ✅ Numeric fields are `number | string | undefined`

**Unit:**
- ✅ Valid inputs pass (at least 3 examples)
- ✅ Invalid inputs fail with ZodError (at least 3 examples)
- ✅ `.passthrough()` allows unknown fields
- ✅ Edge cases: null, undefined, empty string, whitespace

### Required for All Validation Helpers

**Unit:**
- ✅ Success path (valid inputs)
- ✅ Failure path (invalid inputs)
- ✅ Edge cases: null, undefined, NaN, Infinity, empty string, whitespace

**Property-based (for critical helpers only):**
- ✅ Invariant: `toNumber` always returns finite number
- ✅ Invariant: `sanitizeForShell` never allows injection characters
- ✅ Round-trip: `toNumber(String(n))` equals `n` for finite numbers

### Required for All Services Using Validation

**Integration:**
- ✅ Fixture data parses through schema successfully
- ✅ Service methods return validated, typed data
- ✅ Invalid data throws with clear error messages
- ✅ Validation failures logged to observability

## Test Organization

**Structure:**
```
packages/toolkit-validation/
  src/
    hubspot.ts                 # Zod schemas
    utils.ts                   # Helpers (toNumber, sanitizeForShell)
  tests/
    hubspot.test-d.ts          # Type-level tests
    hubspot.test.ts            # Unit tests for schemas
    utils.test.ts              # Unit + property-based tests for helpers
    utils.property.test.ts     # Property-based only (CI-only if slow)

packages/toolkit-hubspot/
  src/
    services/TimesheetsService.ts
  tests/
    integration/
      timesheet-parsing.test.ts  # End-to-end validation tests
```

**Coverage requirements:**
- ✅ Unit tests: 100% line coverage for validation helpers
- ✅ Unit tests: 100% branch coverage for Zod schemas (success + failure paths)
- ✅ Integration tests: All service methods with validation tested
- ✅ Property-based: Critical helpers (`toNumber`, `sanitizeForShell`) have invariant tests

## Integration with Phase 6 Observability

**Test failure tracking:**

All validation test failures logged to TimescaleDB:

```typescript
afterEach(() => {
  if (currentTest.failed) {
    logger.error('Validation test failed', {
      testName: currentTest.name,
      testFile: currentTest.file,
      error: currentTest.error,
      // Vector embedding added by toolkit-logger
    });

    // Logged to test_failures table:
    // - id, timestamp, test_name, test_file, error_message
    // - error_embedding (vector for semantic search)
    // - status ('failed', 'flaky', 'fixed')
  }
});
```

**Phase 7 self-healing integration:**
1. **Detect:** Validation test failures logged to test_failures table
2. **Retrieve:** Vector search finds similar past test failures (semantic similarity on error messages)
3. **Propose:** Agent analyzes failure pattern and suggests schema fix or data normalization
4. **Apply:** Automated PR with fix + tests

**Example self-healing scenario:**
```
Test failure: "Expected number, received string for field 'new_hubspot_field'"
  ↓
Vector search finds similar failure from 3 months ago: "Expected number for billing_rate"
  ↓
Agent proposes: Update schema to accept `z.union([z.number(), z.string()])` for new field
  ↓
PR created with schema update + test
```

## Alternatives Considered

### Single-Layer Testing (Unit Tests Only)

**Pros:** Simpler, one test framework
**Cons:**
- No compile-time type verification
- No edge case discovery (property-based)
- Miss integration issues

**Decision:** Rejected; insufficient coverage

### Snapshot Testing for Schemas

**Pros:** Detects any schema changes
**Cons:**
- Brittle (breaks on legitimate changes)
- Doesn't verify correctness, just consistency
- False positives on valid schema evolution

**Decision:** Rejected; explicit tests better than snapshots

### Manual Testing Only

**Pros:** No test code to maintain
**Cons:**
- No regression protection
- No confidence in changes
- Violates Phase 6 ≥80% coverage requirement

**Decision:** Rejected; automated testing mandatory

### Integration Tests Only

**Pros:** Tests real workflows
**Cons:**
- Slow (setup/teardown overhead)
- Hard to cover edge cases
- Doesn't test type system correctness

**Decision:** Rejected; need multiple layers for comprehensive coverage

## Implementation Timeline

**Week 2:**
- Create toolkit-validation with initial tests (type-level + unit)
- 100% coverage for `toNumber`, `sanitizeForShell`, `isRecord` helpers

**Week 3:**
- Add integration tests to toolkit-hubspot (API → Zod → TypeScript)
- Property-based tests for critical validation logic
- Wire tests into CI pipeline

**Week 4:**
- Add tests for dataops validation
- Integrate with test_failures table (TimescaleDB)
- Verify 100% coverage requirement met

## Success Criteria

- ✅ All Zod schemas have type-level tests (schema inference correctness)
- ✅ All Zod schemas have unit tests (success + failure paths + edge cases)
- ✅ All validation helpers have unit tests (100% line and branch coverage)
- ✅ Critical helpers have property-based tests (`toNumber`, `sanitizeForShell`)
- ✅ All services using validation have integration tests (end-to-end with fixtures)
- ✅ Test failures logged to TimescaleDB with vector embeddings
- ✅ CI enforces coverage thresholds (≥80% for packages, 100% for validation helpers)

## References

- ADR-001: Zod Runtime Validation (why Zod)
- ADR-002: HubSpot Data Modeling (what to validate)
- Phase 6 Tasks: [docs/planning/PHASE6_TASKS.md](../planning/PHASE6_TASKS.md)
- Observability Plan: [docs/planning/observability/IMPLEMENTATION_PLAN.md](../planning/observability/IMPLEMENTATION_PLAN.md) (test_failures table schema)
- Vitest docs: https://vitest.dev
- expect-type: https://github.com/mmkal/expect-type
- fast-check: https://fast-check.dev
