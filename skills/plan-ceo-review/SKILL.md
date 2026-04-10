---
name: plan-ceo-review
description: "Product-level rethinking. 10-star product perspective. Triggers on 'ceo review', 'product review', '제품 리뷰', 'plan review', '계획 리뷰', '10-star', 'rethink'."
context: fork
model: opus
tools: Read, Bash, Glob, Grep
---

# Plan CEO Review

구현 계획을 제품 관점에서 재평가한다. "10-star product" 사고로 사용자 가치를 극대화한다.

**모든 출력은 반드시 한국어로 작성한다.**

---

## Pre-Review
1. 계획 파일 로드 (`.claude/{date}/PLAN-*.md`)
2. 관련 요구사항/이슈 확인

---

## Review Framework

### 1. 사용자 가치 점검
- 이 기능이 실제 사용자 문제를 해결하는가?
- 사용자가 원하는 건가, 우리가 원하는 건가?
- MVP는 무엇인가? (스코프를 줄일 수 있는가?)

### 2. 10-Star 경험
- 1-star: 사용자가 원하는 것을 전혀 할 수 없음
- 5-star: 기본 기대 충족
- 10-star: "이것까지 되네?" (비현실적이어도 됨)
- 목표: 현실적으로 7-8 star 구현

### 3. 스코프 건전성 점검
- 이 PR에서 제거해야 할 것은?
- "이 PR에 포함되지 않는 것" 목록 검토
- 과잉 엔지니어링 신호 확인

### 4. 리스크 평가
- 기술적 리스크
- 일정 리스크
- 의존성 리스크

---

## Output

```markdown
### [대상] CEO 리뷰: [기능명]

**사용자 가치:** [높음/보통/낮음]
**10-Star 평가:** 현재 계획 = X-star, 가능 = Y-star

**유지:**
- [좋은 결정들]

**제거:**
- [스코프에서 빼야 할 것들]

**추가:**
- [빠진 사용자 가치]

**리스크:**
- [주요 리스크와 대응 방안]

**판정:** [GO / REVISE / RETHINK]
```

→ GO: `/{framework}-implement`, REVISE: 계획 수정, RETHINK: `/office-hours`
