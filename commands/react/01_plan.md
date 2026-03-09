# /react:01_plan

Analyze issue and create React-specific implementation plan.

**Agents:** codebase-researcher, framework-docs-researcher, react-feature-library-expert

**Extends:** `/common:plan` (use the same output structure)

---

## React-Specific Planning

On top of the base `/common:plan` workflow, consider these React-specific items:

### Phase 1 추가 분석

1. **Feature Library 배치**
   - 어떤 Feature Library에 속하는가?
   - 새 lib 생성 필요한가? (`npx nx g @nx/js:lib`)
   - 기존 lib에 추가하는 게 맞는가?

2. **데이터 흐름 설계**
   - Component → Hook → TanStack Query → API 경로
   - 어떤 API 엔드포인트가 필요한가?
   - Server state vs Client state 구분

3. **상태 관리 전략**
   - TanStack Query: 서버 데이터 (목록, 상세, CRUD)
   - Zustand: 전역 클라이언트 상태 (UI 상태, 사용자 설정)
   - useState: 로컬 상태만 (폼 입력, 토글)

4. **쿼리키 전략**
   - `createQueryKeys` factory 사용
   - 기존 쿼리키와 충돌 여부
   - Invalidation 전략 (mutation 후 어떤 쿼리를 무효화할 것인가)

### Phase 2 추가 리서치

- 기존 Feature Library 패턴 분석 (codebase-researcher)
- 유사 기능의 hooks/apis 패턴 확인
- Zustand store 구조 확인 (전역 상태 추가 필요 시)

---

## React 전용 Boundaries 추가

### ✅ Do
- Feature Library 구조 (apis/, hooks/, types/, consts/)
- Barrel exports (index.ts) 모든 폴더에
- Component → Hook → API 데이터 흐름
- cn() 유틸리티로 Tailwind 클래스 결합

### ⚠️ Ask First
- 새로운 Zustand store 생성
- 공유 hooks 수정
- 새로운 query key namespace

### 🚫 Don't
- Component에서 직접 API 호출
- useState로 공유 상태 관리
- Barrel export 누락
- 과도한 추상화 (WET > DRY)

---

## React 전용 Checklist 항목

Implementation Checklist에 반드시 포함:

- [ ] Feature Library 생성/확인
- [ ] API 함수 정의 (apis/)
- [ ] Query keys factory 정의 (hooks/)
- [ ] TanStack Query hooks 정의 (hooks/)
- [ ] Type 정의 (types/)
- [ ] Barrel exports 모든 폴더
- [ ] 상태 관리: Zustand(global) / TanStack Query(server) / useState(local)
- [ ] ~200줄 가이드라인, ~5 business props

---

## Review Focus 추가 항목

- [ ] Component → Hook → API 흐름 준수
- [ ] useState가 공유 상태에 사용되지 않음
- [ ] Barrel exports 완전성
- [ ] Query key 충돌 없음
- [ ] cn() 사용으로 Tailwind 클래스 정리
