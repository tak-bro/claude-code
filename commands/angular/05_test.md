# /angular:05_test

Implement and run tests for Angular features.

**Agents:** test-strategy-expert, angular-componentstore-expert

**Extends:** Follows `/angular:02_implement` → `/angular:03_review` → `/angular:04_fix` pipeline output

---

## Pre-Test

**Before writing tests, verify:**
1. ✅ Implementation complete? (`/angular:02_implement`)
2. ✅ Review passed? (`/angular:03_review` → `/angular:04_fix`)
3. ✅ Files to test identified

---

## Angular Test Stack

```
Jest + TestBed                    # Unit/Integration
HttpClientTestingModule           # HTTP mocking
ComponentFixture                  # Component testing
@ngrx/component-store/testing     # Store testing
```

---

## Workflow

### Phase 1: Test Strategy

1. **Analyze implemented files**
   - Store (ComponentStore) → 🔴 Required test
   - Service (API) → 🔴 Required test
   - Page (Smart Component) → 🟡 Integration test
   - Component (Presentational) → 🟢 Optional

2. **Determine mocking targets**
   - API calls → `HttpTestingController`
   - Store → `provideMockStore` or real Store + HTTP mocking
   - Router → `RouterTestingModule`

### Phase 2: Write Tests

#### ComponentStore Test (Required)

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
    httpController.verify(); // Verify no pending requests
  });

  // Initial state
  it('should have initial state', () => { });

  // Effect → API → State change
  it('should load data via effect', () => { });

  // Updater behavior
  it('should update state via updater', () => { });

  // Selector derived value
  it('should compute derived state via selector', () => { });

  // Error handling
  it('should handle API error', () => { });
});
```

#### Page Component Test (Optional)

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

### Phase 3: Run and Verify

```bash
# Test specific files
{pm} test -- --testPathPattern="feature-name"

# With coverage
{pm} test -- --coverage --testPathPattern="feature-name"

# Watch mode
{pm} test -- --watch --testPathPattern="feature-name"
```

---

## Test Writing Rules

### ✅ Do
- Behavior-based test naming (`should [action] when [condition]`)
- AAA pattern (Arrange-Act-Assert)
- Include `destroyed$` cleanup tests
- `httpController.verify()` in `afterEach`
- Test Store registration in Module providers

### 🚫 Don't
- Snapshot tests
- Implementation detail tests (internal method call counts)
- `any` type mocking
- Share state between tests
- `setTimeout` for async waiting → use `fakeAsync`/`tick`

---

## Output

```markdown
### Test Results: [Feature Name]

**Coverage**
| File | Statements | Branches | Functions | Lines |
|------|-----------|----------|-----------|-------|
| `feature.store.ts` | 95% | 90% | 100% | 95% |

**Test Count**
- ✅ Pass: [N]
- ❌ Fail: [N]

**Next Steps**
→ All tests pass → Complete
→ Failures → Run `/angular:04_fix` again
```
