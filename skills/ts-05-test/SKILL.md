---
name: ts-05-test
model: sonnet
description: "TypeScript 테스트 작성 및 실행. Triggers on 'typescript test', 'ts test', '타입스크립트 테스트', 'ts spec'."
tools: Read, Bash, Glob, Grep, Write, Edit
---

# TypeScript Test

**Agents:** test-strategy-expert, tak-typescript-expert

**Maintain 100% of existing test stack and patterns.**

## TypeScript Test Stack

```
Vitest or Jest                    # Test runner (project-dependent)
expectTypeOf (vitest)             # Type testing
tsd                               # Type testing (alternative)
```

---

## Test Strategy

- Utility functions → [Critical] Required
- Services/classes → [Critical] Required
- Types/interfaces → [Important] Type test (if complex)
- Constants/config → [Nice] Optional

---

## Test Patterns

### Utility Function Test
```typescript
describe('calculateTotal', () => {
  it('should sum all item prices', () => {
    const items = [{ price: 100 }, { price: 200 }, { price: 300 }];
    expect(calculateTotal(items)).toBe(600);
  });
  it('should return 0 for empty array', () => { expect(calculateTotal([])).toBe(0); });
  it('should handle negative prices', () => {
    expect(calculateTotal([{ price: 100 }, { price: -50 }])).toBe(50);
  });
});
```

### Error Handling / Result Pattern
```typescript
describe('parseConfig', () => {
  it('should return success for valid config', () => {
    const result = parseConfig(validJson);
    expect(result.success).toBe(true);
  });
  it('should return error for invalid config', () => {
    const result = parseConfig(invalidJson);
    expect(result.success).toBe(false);
  });
});
```

### Type Test (Vitest)
```typescript
import { expectTypeOf } from 'vitest';
describe('type tests', () => {
  it('should infer correct return type', () => {
    expectTypeOf(parseConfig).returns.toEqualTypeOf<Result<Config>>();
  });
});
```

---

## Test Writing Rules

### Do
- Behavior-based naming (`should [action] when [condition]`)
- AAA pattern (Arrange-Act-Assert)
- Edge cases (empty, null, undefined, boundary)
- Test each early return branch
- Result pattern → test both success/error

### 🚫 Don't
- Snapshot tests
- Implementation detail tests
- `any` type mocking
- Share state between tests

---

## Output
(see `_shared/output-templates.md` — Test Output)

→ All pass → Complete / Failures → `/ts-04-fix`
