# /typescript:05_test

Implement and run tests for TypeScript features.

**Agents:** test-strategy-expert, tak-typescript-expert

**Extends:** Follows `/typescript:02_implement` → `/typescript:03_review` → `/typescript:04_fix` pipeline output

---

## Pre-Test

**Before writing tests, verify:**
1. ✅ Implementation complete? (`/typescript:02_implement`)
2. ✅ Review passed? (`/typescript:03_review` → `/typescript:04_fix`)
3. ✅ 테스트 대상 파일 목록 확인

---

## TypeScript Test Stack

```
Vitest 또는 Jest               # Test runner (프로젝트에 따라)
expectTypeOf (vitest)           # 타입 테스트
tsd                             # 타입 테스트 (대안)
```

---

## Workflow

### Phase 1: 테스트 전략 수립

1. **구현된 파일 분석**
   - 유틸리티 함수 → 🔴 필수 테스트
   - 서비스/클래스 → 🔴 필수 테스트
   - 타입/인터페이스 → 🟡 타입 테스트 (복잡한 경우)
   - 상수/설정 → 🟢 선택

2. **모킹 대상 결정**
   - 외부 API → mock 함수
   - 파일시스템 → mock
   - 날짜/타이머 → `vi.useFakeTimers()` / `jest.useFakeTimers()`

### Phase 2: 테스트 작성

#### 유틸리티 함수 테스트 (필수)

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

#### 에러 핸들링 테스트

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

#### 타입 테스트 (복잡한 제네릭/유틸리티 타입)

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

#### 비동기 함수 테스트

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

### Phase 3: 실행 및 검증

```bash
# Vitest
{pm} vitest run --reporter=verbose src/utils/feature-name

# Jest
{pm} jest --verbose --testPathPattern="feature-name"

# 커버리지 포함
{pm} vitest run --coverage src/utils/feature-name

# Watch 모드
{pm} vitest watch src/utils/feature-name
```

---

## 테스트 작성 규칙

### ✅ Do
- 행동 기반 테스트 네이밍 (`should [action] when [condition]`)
- AAA 패턴 (Arrange-Act-Assert)
- Edge case 포함 (빈 배열, null, undefined, 경계값)
- Early return 로직 각 분기 테스트
- Result 패턴 → success/error 양쪽 테스트

### 🚫 Don't
- 구현 상세 테스트
- `any` 타입 모킹
- 테스트 간 상태 공유
- 외부 의존성 직접 호출 (모킹 필수)
- 하드코딩된 매직 넘버 (상수 사용)

---

## Output

```markdown
### 🧪 Test Results: [Feature Name]

**커버리지**
| 파일 | Statements | Branches | Functions | Lines |
|------|-----------|----------|-----------|-------|
| `feature-util.ts` | 95% | 90% | 100% | 95% |

**테스트 수**
- ✅ Pass: [N]
- ❌ Fail: [N]

**다음 단계**
→ 모든 테스트 통과 시 완료
→ 실패 시 `/typescript:04_fix` 재실행
```
