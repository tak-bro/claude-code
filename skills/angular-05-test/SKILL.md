---
name: angular-05-test
description: "Angular test writing and execution. Triggers on 'angular test', '앵귤러 테스트', 'ng test', 'angular spec', 'angular 유닛 테스트'."
tools: Read, Bash, Glob, Grep, Write, Edit
model: sonnet

# Angular Test

Write and run tests for Angular features.

**Agents:** test-strategy-expert, angular-componentstore-expert

## Pre-Test

1. Implementation complete? (`/angular-02-implement`)
2. Review passed? (`/angular-03-review` → `/angular-04-fix`)
3. Files to test identified

---

## Angular Test Stack

```
Jest + TestBed                    # Unit/Integration
HttpClientTestingModule           # HTTP mocking
ComponentFixture                  # Component testing
@ngrx/component-store/testing     # Store testing
```

**Maintain 100% compatibility with existing test stack and patterns.**

---

## Test Strategy

1. **Analyze implemented files**
   - Store (ComponentStore) → [Critical] Required test
   - Service (API) → [Critical] Required test
   - Page (Smart Component) → [Important] Integration test
   - Component (Presentational) → [Nice] Optional

2. **Determine mocking targets**
   - API calls → `HttpTestingController`
   - Store → `provideMockStore` or real Store + HTTP mocking
   - Router → `RouterTestingModule`

---

## Test Patterns

### ComponentStore Test (Required)

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
    httpController.verify();
  });

  it('should have initial state', () => { });
  it('should load data via effect', () => { });
  it('should update state via updater', () => { });
  it('should compute derived state via selector', () => { });
  it('should handle API error', () => { });
});
```

### Page Component Test (Optional)

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

---

## Test Writing Rules

### Do
- Behavior-based test naming (`should [action] when [condition]`)
- AAA pattern (Arrange-Act-Assert) required
- Include `destroyed$` cleanup tests
- `httpController.verify()` in `afterEach`
- Test Store registration in Module providers

### 🚫 Don't
- Snapshot tests
- Implementation detail tests
- `any` type mocking
- Share state between tests
- `setTimeout` → use `fakeAsync`/`tick`

---

## Run and Verify

```bash
{pm} test -- --testPathPattern="feature-name"
{pm} test -- --coverage --testPathPattern="feature-name"
```

---

## Output

```markdown
### Test Results: [Feature Name]

**Coverage**
| File | Statements | Branches | Functions | Lines |
|------|-----------|----------|-----------|-------|

**Test Count**
- Pass: [N]
- Fail: [N]

→ All tests pass → Complete
→ Failures → Run `/angular-04-fix` again
```
