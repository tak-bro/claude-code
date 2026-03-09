# /typescript:01_plan

Analyze issue and create TypeScript-specific implementation plan.

**Agents:** codebase-researcher, framework-docs-researcher, tak-typescript-expert

**Extends:** `/common:plan` (use the same output structure)

---

## TypeScript-Specific Planning

On top of the base `/common:plan` workflow, consider these TypeScript-specific items:

### Phase 1 추가 분석

1. **타입 설계**
   - 핵심 도메인 타입/인터페이스 정의
   - Discriminated unions 필요 여부 (Result<T>, 상태 머신 등)
   - 유틸리티 타입 활용 (Partial, Pick, Omit, Record 등)
   - 제네릭 필요 여부 및 수준

2. **모듈 구조**
   - 어떤 모듈/폴더에 배치하는가?
   - Barrel export (index.ts) 계획
   - 기존 모듈과의 의존 관계

3. **에러 핸들링 전략**
   - Result 패턴 vs try-catch
   - null/undefined 처리 방식 (early return)
   - 에러 타입 정의

### Phase 2 추가 리서치

- 기존 타입 패턴 분석 (codebase-researcher)
- 유사 기능의 유틸리티/서비스 확인
- 공유 타입 정의 확인 (중복 방지)

---

## TypeScript 전용 Boundaries 추가

### ✅ Do
- const + arrow function
- Named exports only
- Early returns로 flat 구조
- 복잡한 조건 → named variables
- 명확한 타입 정의

### ⚠️ Ask First
- 새로운 유틸리티 타입 생성
- 공유 모듈 수정
- 제네릭 3단계 이상 중첩

### 🚫 Don't
- `export default`
- `function` keyword
- `any` without justification
- Deep nesting
- 기존 코드 불필요한 복잡화

---

## TypeScript 전용 Checklist 항목

Implementation Checklist에 반드시 포함:

- [ ] 도메인 타입/인터페이스 정의
- [ ] const + arrow function
- [ ] Named exports + barrel exports
- [ ] null/undefined 처리 (early returns)
- [ ] 복잡한 조건 → named variables
- [ ] 테스트 가능한 구조
- [ ] 5-second naming rule 통과
- [ ] 기존 코드 복잡화 없음

---

## Review Focus 추가 항목

- [ ] 타입 안정성 (any 없음 또는 정당화)
- [ ] Early returns로 flat 구조
- [ ] Named variables로 가독성 확보
- [ ] 기존 패턴과 일관성
- [ ] 테스트 가능 구조
