# /angular:01_plan

Analyze issue and create Angular-specific implementation plan.

**Agents:** codebase-researcher, framework-docs-researcher, angular-componentstore-expert

**Extends:** `/common:plan` (use the same output structure)

---

## Angular-Specific Planning

On top of the base `/common:plan` workflow, consider these Angular-specific items:

### Phase 1 추가 분석

1. **모듈 구조 파악**
   - 어떤 Feature Module에 속하는가?
   - 새 모듈 생성이 필요한가?
   - SharedModule에서 필요한 것은?

2. **라우팅 설계**
   - 라우팅 모듈에 등록할 경로
   - Guard 순서: App → Auth → Feature
   - Lazy loading 적용 여부

3. **ComponentStore 설계**
   - State interface 정의
   - 필요한 Selectors, Updaters, Effects
   - Message pattern 타입 정의

### Phase 2 추가 리서치

- 기존 ComponentStore 패턴 분석 (codebase-researcher)
- Ionic 프로젝트인 경우 IonBasePage 패턴 확인
- TanStack Query 사용 여부 및 기존 QueryService 패턴 확인

---

## Angular 전용 Boundaries 추가

### ✅ Do
- Module-based architecture (standalone 금지)
- ComponentStore를 Module providers에 등록
- Page(smart) / Component(presentational) 분리 계획
- Message pattern 포함한 State 설계

### ⚠️ Ask First
- 새로운 Shared 서비스 생성
- Core state-manager 수정
- Guard 순서 변경

### 🚫 Don't
- Standalone component 사용
- Component에서 직접 API 호출 계획
- Store를 Component providers에 등록

---

## Angular 전용 Checklist 항목

Implementation Checklist에 반드시 포함:

- [ ] Feature Module 생성/수정
- [ ] Routing Module 등록
- [ ] ComponentStore state interface 정의
- [ ] Message pattern 타입 정의
- [ ] Page/Component 분리
- [ ] (Ionic) IonBasePage extends 필요 여부
- [ ] (Ionic) IonicRouteStrategy 확인
- [ ] (TanStack Query) Query keys factory 필요 여부

---

## Review Focus 추가 항목

- [ ] Module providers에 Store 등록 여부
- [ ] destroyed$ cleanup 패턴
- [ ] Presentational component에 Store 주입 없음
- [ ] (Ionic) ionViewWillEnter lifecycle 사용
