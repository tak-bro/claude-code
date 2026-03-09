# /common:debug

에러 메시지나 스택 트레이스를 분석하고 해결 방안 제시.

**Agents:** codebase-researcher, framework-docs-researcher

---

## Workflow

### Phase 1: 에러 분석

1. **에러 메시지 파싱**
   - 에러 타입 (TypeError, ReferenceError, HttpError, etc.)
   - 발생 위치 (파일, 라인, 컬럼)
   - 스택 트레이스 해석

2. **에러 분류**

   | 유형 | 예시 | 접근법 |
   |------|------|--------|
   | 🔴 런타임 에러 | TypeError, null reference | 스택 트레이스 → 소스 코드 추적 |
   | 🟡 빌드 에러 | TypeScript, Webpack, Vite | 에러 메시지 → 설정/타입 확인 |
   | 🟢 린트 에러 | ESLint, Prettier | 규칙 확인 → 코드 수정 |
   | 🔵 테스트 에러 | Jest, Vitest assertion | 기대값 vs 실제값 비교 |
   | 🟣 네트워크 에러 | CORS, 401, 500 | API 설정/인증 확인 |

### Phase 2: 관련 코드 탐색 🔀

```bash
# 병렬 실행 (Task 도구 활용)
Lane 1: codebase-researcher        # 에러 발생 파일 및 관련 코드 분석
Lane 2: framework-docs-researcher  # 에러 관련 공식 문서 조사 (필요 시)
```

**codebase-researcher 탐색 범위:**
- 에러 발생 파일
- 에러 발생 함수의 호출자 (call chain)
- 관련 타입 정의
- 유사 패턴 사용 파일 (정상 동작하는 코드)

**framework-docs-researcher 조사 대상 (해당 시):**
- 에러 메시지 관련 공식 문서
- Breaking changes (버전 업그레이드 후 에러인 경우)
- Known issues / GitHub issues

### Phase 3: 원인 추론

1. **직접 원인** → 에러를 발생시키는 코드
2. **근본 원인** → 왜 그 코드가 문제인지
3. **관련 요인** → 환경, 설정, 의존성 등

### Phase 4: 수정 방안 제시

---

## Output Structure

```markdown
### 🔍 Debug Report: [Error Summary]

---

**에러 요약**
- 유형: [런타임/빌드/린트/테스트/네트워크]
- 메시지: `[에러 메시지]`
- 위치: `[파일:라인]`

---

**원인 분석**

| | 설명 |
|---|------|
| 직접 원인 | [에러를 발생시키는 코드/상황] |
| 근본 원인 | [왜 그 코드가 문제인지] |
| 관련 요인 | [환경, 설정, 의존성 등] |

---

**수정 방안**

✅ 추천 수정:
[코드 변경 내용 + 이유]

⚠️ 대안:
[다른 접근법이 있는 경우]

---

**검증 방법**
```bash
# 수정 후 확인 명령어
{pm} run build
{pm} test -- --testPathPattern="affected-file"
```

---

**재발 방지**
- [동일 유형 에러 방지를 위한 제안]
```

---

## 흔한 에러 패턴 참조

### TypeScript

| 에러 | 흔한 원인 | 해결 |
|------|----------|------|
| `TS2322: Type 'X' is not assignable to 'Y'` | 타입 불일치 | 타입 좁히기, 타입 가드 |
| `TS2345: Argument of type 'X' is not assignable` | 함수 인자 타입 불일치 | 인자 타입 확인 |
| `TS7006: Parameter implicitly has 'any' type` | 타입 미선언 | 명시적 타입 추가 |
| `TS2339: Property 'X' does not exist` | 없는 속성 접근 | 타입 정의 확인 |

### Angular

| 에러 | 흔한 원인 | 해결 |
|------|----------|------|
| `NullInjectorError: No provider for X` | DI 미등록 | Module providers에 추가 |
| `ExpressionChangedAfterItHasBeenCheckedError` | CD 중 값 변경 | 비동기 처리, OnPush |
| `Can't bind to 'X' since it isn't a known property` | Module import 누락 | 해당 모듈 import |

### React

| 에러 | 흔한 원인 | 해결 |
|------|----------|------|
| `Too many re-renders` | 무한 리렌더링 루프 | useEffect deps 확인 |
| `Cannot update during render` | 렌더 중 상태 변경 | useEffect로 이동 |
| `Invalid hook call` | Hook 규칙 위반 | 조건문 밖으로 이동 |
| `Hydration mismatch` | SSR/CSR 불일치 | 동적 콘텐츠 처리 |
