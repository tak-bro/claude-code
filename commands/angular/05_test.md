# /angular:05_test

Implement and run tests for Angular features.

**Agents:** test-strategy-expert, angular-componentstore-expert

**Extends:** Follows `/angular:02_implement` → `/angular:03_review` → `/angular:04_fix` pipeline output

---

## Pre-Test

**Before writing tests, verify:**
1. ✅ Implementation complete? (`/angular:02_implement`)
2. ✅ Review passed? (`/angular:03_review` → `/angular:04_fix`)
3. ✅ 테스트 대상 파일 목록 확인

---

## Angular Test Stack

```
Jest + TestBed                    # Unit/Integration
HttpClientTestingModule           # HTTP 모킹
ComponentFixture                  # Component 테스트
@ngrx/component-store/testing     # Store 테스트
```

---

## Workflow

### Phase 1: 테스트 전략 수립

1. **구현된 파일 분석**
   - Store (ComponentStore) → 🔴 필수 테스트
   - Service (API) → 🔴 필수 테스트
   - Page (Smart Component) → 🟡 통합 테스트
   - Component (Presentational) → 🟢 선택

2. **모킹 대상 결정**
   - API 호출 → `HttpTestingController`
   - Store → `provideMockStore` 또는 실제 Store + HTTP 모킹
   - Router → `RouterTestingModule`

### Phase 2: 테스트 작성

#### ComponentStore 테스트 (필수)

```typescript
describe('FeatureStore', () => {
  let store: FeatureStore;
  let httpController: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [FeatureStore],
    });
    store = TestBed.inject(FeatureStore);
    httpController = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpController.verify(); // 미처리 요청 확인
  });

  // State 초기값
  it('should have initial state', () => { });

  // Effect → API → State 변경
  it('should load data via effect', () => { });

  // Updater 동작
  it('should update state via updater', () => { });

  // Selector 파생값
  it('should compute derived state via selector', () => { });

  // 에러 처리
  it('should handle API error', () => { });
});
```

#### Page Component 테스트 (선택)

```typescript
describe('FeaturePage', () => {
  let component: FeaturePageComponent;
  let fixture: ComponentFixture<FeaturePageComponent>;
  let store: FeatureStore;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [FeaturePageComponent],
      providers: [FeatureStore],
      imports: [HttpClientTestingModule],
    }).compileComponents();

    fixture = TestBed.createComponent(FeaturePageComponent);
    component = fixture.componentInstance;
    store = TestBed.inject(FeatureStore);
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

### Phase 3: 실행 및 검증

```bash
# 특정 파일 테스트
{pm} test -- --testPathPattern="feature-name"

# 커버리지 포함
{pm} test -- --coverage --testPathPattern="feature-name"

# Watch 모드
{pm} test -- --watch --testPathPattern="feature-name"
```

---

## 테스트 작성 규칙

### ✅ Do
- 행동 기반 테스트 네이밍 (`should [action] when [condition]`)
- AAA 패턴 (Arrange-Act-Assert)
- `destroyed$` cleanup 테스트 포함
- `afterEach`에서 `httpController.verify()`
- Module providers에 Store 등록 확인 테스트

### 🚫 Don't
- 스냅샷 테스트
- 구현 상세 테스트 (내부 메서드 호출 횟수)
- `any` 타입 모킹
- 테스트 간 상태 공유
- `setTimeout`으로 비동기 대기 → `fakeAsync`/`tick` 사용

---

## Output

```markdown
### 🧪 Test Results: [Feature Name]

**커버리지**
| 파일 | Statements | Branches | Functions | Lines |
|------|-----------|----------|-----------|-------|
| `feature.store.ts` | 95% | 90% | 100% | 95% |

**테스트 수**
- ✅ Pass: [N]
- ❌ Fail: [N]

**다음 단계**
→ 모든 테스트 통과 시 완료
→ 실패 시 `/angular:04_fix` 재실행
```
