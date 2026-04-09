---
name: review-pr
description: "GitHub PR 코드 리뷰. Triggers on 'review pr', 'PR 리뷰', 'pull request review', 'pr review', 'PR 검토', 'code review pr'."
context: fork
tools: Read, Bash, Glob, Grep
---

# Review PR

다른 사람의 Pull Request를 리뷰하고 피드백을 제공한다.

**Context: fork** — 분리 컨텍스트에서 실행.

---

## Usage

```
/review-pr 123              ← PR 번호
/review-pr                   ← 현재 브랜치의 PR
/review-pr https://github.com/owner/repo/pull/123
```

---

## Phase 1: PR 정보 수집

```bash
# PR 번호로 정보 가져오기
gh pr view {PR_NUMBER} --json title,body,files,additions,deletions,baseRefName,headRefName
gh pr diff {PR_NUMBER}
gh pr checks {PR_NUMBER}
```

- PR 제목, 설명, 변경 파일 목록
- base branch ← head branch
- CI 상태 확인

---

## Phase 2: 변경 범위 분석

1. **영향 분석**: 변경된 파일 → 영향받는 모듈/기능
2. **변경 크기 판단**: S/M/L → 리뷰 깊이 결정
3. **리스크 영역 식별**: API 변경, 상태 관리, 보안 관련 코드

---

## Phase 3: 코드 리뷰

### [Critical] (반드시 수정)
- 버그 또는 로직 오류
- 보안 취약점 (인젝션, 하드코딩 시크릿, 인증 우회)
- 데이터 손실 가능성
- 기존 기능 파괴 (breaking change 미문서화)

### [Important] (권장)
- 프로젝트 컨벤션 위반 (named export, arrow function 등)
- 테스트 누락
- 에러 핸들링 부족
- 성능 우려

### [Nice-to-have] (선택)
- 네이밍 개선
- 코드 간소화
- 문서 추가

---

## Phase 4: 자동 검증

```bash
# 로컬에서 PR 체크아웃
gh pr checkout {PR_NUMBER}

# 빌드/테스트 (프로젝트에 따라)
{pm} install
{pm} run lint
{pm} test
{pm} run build
```

---

## Phase 5: 리뷰 결과 생성

```markdown
## PR Review: #{PR_NUMBER} — {Title}

**변경 범위:** {files}개 파일 (+{additions}/-{deletions})
**리스크:** Low / Medium / High

### [Critical]
- **{file}:{line}** — {문제 설명}
  ```suggestion
  {수정 제안 코드}
  ```

### [Important]
- **{file}:{line}** — {문제 설명}

### [Nice-to-have]
- {제안}

### 테스트 결과
- Build: PASS/FAIL
- Lint: PASS/FAIL
- Test: PASS/FAIL ({N} tests)

### 결론
- [ ] Approve — 이슈 없음
- [ ] Request Changes — Critical 이슈 있음
- [ ] Comment — 논의 필요
```

---

## 주의사항

- PR 설명에 "현재 동작"만 기술. "혁신적", "획기적" 같은 마케팅 용어 사용 금지
- 리뷰 코멘트는 구체적으로. "이상하다" 대신 "X 때문에 Y 문제가 생길 수 있다"
- 사용자에게 직접 `gh pr review` 실행 여부를 확인 후 진행
