# /react:05_test

Implement and run tests for React features.

**Agents:** test-strategy-expert, react-feature-library-expert

**Extends:** Follows `/react:02_implement` → `/react:03_review` → `/react:04_fix` pipeline output

---

## Pre-Test

**Before writing tests, verify:**
1. ✅ Implementation complete? (`/react:02_implement`)
2. ✅ Review passed? (`/react:03_review` → `/react:04_fix`)
3. ✅ Files to test identified

---

## React Test Stack

```
Vitest                            # Test runner
React Testing Library             # Component testing
MSW (Mock Service Worker)         # API mocking
@testing-library/react → renderHook  # Hook testing
```

---

## Workflow

### Phase 1: Test Strategy

1. **Analyze implemented files**
   - API functions (apis/) → 🔴 Required test
   - Query/Mutation hooks (hooks/) → 🔴 Required test
   - Custom hooks (hooks/) → 🔴 Required test
   - Container component → 🟡 Integration test
   - Presentational component → 🟢 Optional

2. **Determine mocking targets**
   - API calls → MSW handlers
   - TanStack Query → `QueryClientProvider` wrapper
   - Zustand → Store reset in `beforeEach`
   - Router → `MemoryRouter`

### Phase 2: Write Tests

#### TanStack Query Hook Test (Required)

```typescript
const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

describe('useFeatureData', () => {
  it('should fetch data successfully', async () => {
    const { result } = renderHook(() => useFeatureData(1), {
      wrapper: createWrapper(),
    });

    await waitFor(() => expect(result.current.isSuccess).toBe(true));
    expect(result.current.data).toEqual(mockData);
  });

  it('should handle error', async () => {
    server.use(
      http.get('/api/feature/:id', () => {
        return HttpResponse.json({ message: 'Not Found' }, { status: 404 });
      }),
    );

    const { result } = renderHook(() => useFeatureData(999), {
      wrapper: createWrapper(),
    });

    await waitFor(() => expect(result.current.isError).toBe(true));
  });
});
```

#### MSW Setup

```typescript
// mocks/handlers.ts
export const handlers = [
  http.get('/api/feature', () => {
    return HttpResponse.json({ data: mockFeatureList });
  }),
  http.get('/api/feature/:id', ({ params }) => {
    return HttpResponse.json({ data: mockFeature });
  }),
  http.post('/api/feature', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ data: { id: 1, ...body } }, { status: 201 });
  }),
];

// mocks/server.ts
export const server = setupServer(...handlers);

// setup
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

#### Zustand Store Test

```typescript
describe('useFeatureStore', () => {
  beforeEach(() => {
    // Reset store
    useFeatureStore.setState(initialState);
  });

  it('should update filter', () => {
    const { result } = renderHook(() => useFeatureStore());
    act(() => result.current.setFilter({ status: 'active' }));
    expect(result.current.filter).toEqual({ status: 'active' });
  });
});
```

#### Component Test (Optional)

```typescript
describe('FeatureList', () => {
  it('should render items', async () => {
    render(
      <QueryClientProvider client={queryClient}>
        <MemoryRouter>
          <FeatureList />
        </MemoryRouter>
      </QueryClientProvider>,
    );

    await waitFor(() => {
      expect(screen.getByText('Feature Item 1')).toBeInTheDocument();
    });
  });

  it('should show loading state', () => {
    render(/* ... */);
    expect(screen.getByRole('progressbar')).toBeInTheDocument();
  });
});
```

### Phase 3: Run and Verify

```bash
# Test specific files
{pm} vitest run --reporter=verbose src/features/feature-name

# With coverage
{pm} vitest run --coverage src/features/feature-name

# Watch mode
{pm} vitest watch src/features/feature-name
```

---

## Test Writing Rules

### ✅ Do
- Behavior-based test naming (`should [action] when [condition]`)
- AAA pattern (Arrange-Act-Assert)
- `waitFor` for async operations
- MSW for network layer mocking
- Fresh `QueryClient` per test

### 🚫 Don't
- Snapshot tests
- Implementation detail tests (internal state, call counts)
- `any` type mocking
- Share `QueryClient` between tests
- `setTimeout` for async waiting → use `waitFor`

---

## Output

```markdown
### Test Results: [Feature Name]

**Coverage**
| File | Statements | Branches | Functions | Lines |
|------|-----------|----------|-----------|-------|
| `use-feature.ts` | 95% | 90% | 100% | 95% |

**Test Count**
- ✅ Pass: [N]
- ❌ Fail: [N]

**Next Steps**
→ All tests pass → Complete
→ Failures → Run `/react:04_fix` again
```
