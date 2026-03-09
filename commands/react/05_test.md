# /react:05_test

Implement and run tests for React features.

**Agents:** test-strategy-expert, react-feature-library-expert

**Extends:** Follows `/react:02_implement` → `/react:03_review` → `/react:04_fix` pipeline output

---

## Pre-Test

**Before writing tests, verify:**
1. ✅ Implementation complete? (`/react:02_implement`)
2. ✅ Review passed? (`/react:03_review` → `/react:04_fix`)
3. ✅ 테스트 대상 파일 목록 확인

---

## React Test Stack

```
Vitest                            # Test runner
React Testing Library             # Component 테스트
MSW (Mock Service Worker)         # API 모킹
@testing-library/react → renderHook  # Hook 테스트
```

---

## Workflow

### Phase 1: 테스트 전략 수립

1. **구현된 파일 분석**
   - API 함수 (apis/) → 🔴 필수 테스트
   - Query/Mutation hooks (hooks/) → 🔴 필수 테스트
   - Custom hooks (hooks/) → 🔴 필수 테스트
   - Container component → 🟡 통합 테스트
   - Presentational component → 🟢 선택

2. **모킹 대상 결정**
   - API 호출 → MSW handlers
   - TanStack Query → `QueryClientProvider` wrapper
   - Zustand → Store reset in `beforeEach`
   - Router → `MemoryRouter`

### Phase 2: 테스트 작성

#### TanStack Query Hook 테스트 (필수)

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

#### Zustand Store 테스트

```typescript
describe('useFeatureStore', () => {
  beforeEach(() => {
    // Store 초기화
    useFeatureStore.setState(initialState);
  });

  it('should update filter', () => {
    const { result } = renderHook(() => useFeatureStore());
    act(() => result.current.setFilter({ status: 'active' }));
    expect(result.current.filter).toEqual({ status: 'active' });
  });
});
```

#### Component 테스트 (선택)

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

### Phase 3: 실행 및 검증

```bash
# 특정 파일 테스트
{pm} vitest run --reporter=verbose src/features/feature-name

# 커버리지 포함
{pm} vitest run --coverage src/features/feature-name

# Watch 모드
{pm} vitest watch src/features/feature-name
```

---

## 테스트 작성 규칙

### ✅ Do
- 행동 기반 테스트 네이밍 (`should [action] when [condition]`)
- AAA 패턴 (Arrange-Act-Assert)
- `waitFor`로 비동기 대기
- MSW로 네트워크 레이어 모킹
- 매 테스트마다 fresh `QueryClient` 생성

### 🚫 Don't
- 스냅샷 테스트
- 구현 상세 테스트 (내부 상태, 호출 횟수)
- `any` 타입 모킹
- 테스트 간 `QueryClient` 공유
- `setTimeout`으로 비동기 대기 → `waitFor` 사용

---

## Output

```markdown
### 🧪 Test Results: [Feature Name]

**커버리지**
| 파일 | Statements | Branches | Functions | Lines |
|------|-----------|----------|-----------|-------|
| `use-feature.ts` | 95% | 90% | 100% | 95% |

**테스트 수**
- ✅ Pass: [N]
- ❌ Fail: [N]

**다음 단계**
→ 모든 테스트 통과 시 완료
→ 실패 시 `/react:04_fix` 재실행
```
