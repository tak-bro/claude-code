---
name: react-05-test
description: "React 테스트 작성 및 실행. Triggers on 'react test', '리액트 테스트', 'react spec', 'react 유닛 테스트'."
tools: Read, Bash, Glob, Grep, Write, Edit
---

# React Test

**Agents:** test-strategy-expert, react-feature-library-expert

**Maintain 100% of existing test stack and patterns.**

## React Test Stack

```
Vitest                            # Test runner
React Testing Library             # Component testing
MSW (Mock Service Worker)         # API mocking
@testing-library/react → renderHook  # Hook testing
```

---

## Test Strategy

- API functions (apis/) → [Critical] Required
- Query/Mutation hooks (hooks/) → [Critical] Required
- Custom hooks → [Critical] Required
- Container component → [Important] Integration test
- Presentational component → [Nice] Optional

---

## Test Patterns

### TanStack Query Hook Test

```typescript
const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
};

describe('useFeatureData', () => {
  it('should fetch data successfully', async () => {
    const { result } = renderHook(() => useFeatureData(1), { wrapper: createWrapper() });
    await waitFor(() => expect(result.current.isSuccess).toBe(true));
    expect(result.current.data).toEqual(mockData);
  });
});
```

### MSW Setup

```typescript
export const handlers = [
  http.get('/api/feature', () => HttpResponse.json({ data: mockFeatureList })),
  http.post('/api/feature', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ data: { id: 1, ...body } }, { status: 201 });
  }),
];

export const server = setupServer(...handlers);
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

### Zustand Store Test

```typescript
describe('useFeatureStore', () => {
  beforeEach(() => { useFeatureStore.setState(initialState); });
  it('should update filter', () => {
    const { result } = renderHook(() => useFeatureStore());
    act(() => result.current.setFilter({ status: 'active' }));
    expect(result.current.filter).toEqual({ status: 'active' });
  });
});
```

---

## Test Writing Rules

### Do
- Behavior-based naming (`should [action] when [condition]`)
- AAA pattern (Arrange-Act-Assert)
- `waitFor` for async operations
- MSW for network mocking
- Fresh `QueryClient` per test

### 🚫 Don't
- Snapshot tests
- Implementation detail tests
- `any` type mocking
- Share `QueryClient` between tests

---

## Output
(see `_shared/output-templates.md` — Test Output)

→ All pass → Complete / Failures → `/react-04-fix`
