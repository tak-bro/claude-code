# /typescript:05_test

Implement and run tests for TypeScript features.

**Agents:** test-strategy-expert, tak-typescript-expert

**Extends:** Follows `/typescript:02_implement` → `/typescript:03_review` → `/typescript:04_fix` pipeline output

---

## Pre-Test

**Before writing tests, verify:**
1. ✅ Implementation complete? (`/typescript:02_implement`)
2. ✅ Review passed? (`/typescript:03_review` → `/typescript:04_fix`)
3. ✅ Files to test identified

---

## TypeScript Test Stack

```
Vitest or Jest                    # Test runner (project-dependent)
expectTypeOf (vitest)             # Type testing
tsd                               # Type testing (alternative)
```

---

## Workflow

### Phase 1: Test Strategy

1. **Analyze implemented files**
   - Utility functions → 🔴 Required test
   - Services/classes → 🔴 Required test
   - Types/interfaces → 🟡 Type test (if complex)
   - Constants/config → 🟢 Optional

2. **Determine mocking targets**
   - External API → mock functions
   - File system → mock
   - Date/timers → `vi.useFakeTimers()` / `jest.useFakeTimers()`

### Phase 2: Write Tests

#### Utility Function Test (Required)

```typescript
describe('calculateTotal', () => {
  it('should sum all item prices', () => {
    const items = [{ price: 100 }, { price: 200 }, { price: 300 }];
    expect(calculateTotal(items)).toBe(600);
  });

  it('should return 0 for empty array', () => {
    expect(calculateTotal([])).toBe(0);
  });

  it('should handle negative prices', () => {
    const items = [{ price: 100 }, { price: -50 }];
    expect(calculateTotal(items)).toBe(50);
  });
});
```

#### Error Handling Test

```typescript
describe('parseConfig', () => {
  it('should return success result for valid config', () => {
    const result = parseConfig(validJson);
    expect(result.success).toBe(true);
    if (result.success) {
      expect(result.data.port).toBe(3000);
    }
  });

  it('should return error result for invalid config', () => {
    const result = parseConfig(invalidJson);
    expect(result.success).toBe(false);
    if (!result.success) {
      expect(result.error.code).toBe('INVALID_CONFIG');
    }
  });
});
```

#### Type Test (for complex generics/utility types)

```typescript
// Vitest expectTypeOf
import { expectTypeOf } from 'vitest';

describe('type tests', () => {
  it('should infer correct return type', () => {
    expectTypeOf(parseConfig).returns.toEqualTypeOf<Result<Config>>();
  });

  it('should accept readonly arrays', () => {
    expectTypeOf(calculateTotal).parameter(0).toEqualTypeOf<readonly { price: number }[]>();
  });
});
```

#### Async Function Test

```typescript
describe('fetchWithRetry', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('should retry on failure', async () => {
    const mockFn = vi.fn()
      .mockRejectedValueOnce(new Error('fail'))
      .mockResolvedValueOnce({ data: 'ok' });

    const promise = fetchWithRetry(mockFn, { maxRetries: 3 });
    await vi.advanceTimersByTimeAsync(1000);

    const result = await promise;
    expect(result).toEqual({ data: 'ok' });
    expect(mockFn).toHaveBeenCalledTimes(2);
  });
});
```

### Phase 3: Run and Verify

```bash
# Vitest
{pm} vitest run --reporter=verbose src/utils/feature-name

# Jest
{pm} jest --verbose --testPathPattern="feature-name"

# With coverage
{pm} vitest run --coverage src/utils/feature-name

# Watch mode
{pm} vitest watch src/utils/feature-name
```

---

## Test Writing Rules

### ✅ Do
- Behavior-based test naming (`should [action] when [condition]`)
- AAA pattern (Arrange-Act-Assert)
- Include edge cases (empty array, null, undefined, boundary values)
- Test each branch of early return logic
- Result pattern → test both success/error

### 🚫 Don't
- Implementation detail tests
- `any` type mocking
- Share state between tests
- Direct external dependency calls (mock required)
- Hard-coded magic numbers (use constants)

---

## Output

```markdown
### Test Results: [Feature Name]

**Coverage**
| File | Statements | Branches | Functions | Lines |
|------|-----------|----------|-----------|-------|
| `feature-util.ts` | 95% | 90% | 100% | 95% |

**Test Count**
- ✅ Pass: [N]
- ❌ Fail: [N]

**Next Steps**
→ All tests pass → Complete
→ Failures → Run `/typescript:04_fix` again
```
