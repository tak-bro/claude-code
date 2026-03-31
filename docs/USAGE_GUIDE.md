# 사용 가이드
> 실제 사용 시나리오 중심 — 따라하면서 익히기

---

## 1. 새 프로젝트 처음 열었을 때

```
You: /generate-codebase-context

Claude: [프로젝트 전체 스캔]
        .claude/llms.txt 생성 완료
        - 17개 모듈, 3개 공유 라이브러리
        - Angular 18 + Ionic 7 + NgRx ComponentStore
        → 다음: /explore
```

```
You: /explore
     인증 모듈 구조 파악해줘.

Claude: ## 탐색: auth 모듈
        modules/auth/ (pages 3, stores 2, services 1)
        패턴: ComponentStore + Message Pattern
        의존성: core/managers/token.manager.ts
        → 다음: /{fw}-01-plan
```

---

## 2. 새 기능 개발 (M 사이즈)

### Step 1: 계획

plan 스킬은 Write/Edit 도구가 없어서 코드 수정 없이 계획만 세운다. (Plan Mode 별도 진입 불필요)

```
You: /angular-01-plan
     인증 모듈에 소셜 로그인(Google, Apple) 추가.

Claude: ## Phase 0.5: Rethink
        "소셜 로그인"을 요청했는데, 기존 이메일 로그인과
        통합 계정이 필요한가요? 아니면 별도?

You: 통합. 같은 이메일이면 기존 계정에 연결.

Claude: Plan: 소셜 로그인
        Phase 1: SocialAuthStore + Google/Apple SDK
        Phase 2: SocialLoginPage extends IonBasePage
        Phase 3: AccountLinkingService (이메일 기반 통합)

        What's NOT in This PR:
        - Facebook 로그인 (V2)
        - 소셜 프로필 사진 동기화

        .claude/20260330/PLAN-143022.md 저장됨
```

think hard가 필요한 경우:
```
You: think hard
     SocialAuthStore 상태 구조를 어떻게 잡을지 고민해봐.

Claude: [깊은 분석]
        Option A: 단일 Store (provider별 state 분기)
        Option B: ProviderStore 분리 (GoogleStore, AppleStore)
        → A 추천. provider가 2개뿐이고 flow가 동일.
```

### Step 2: Auto-Accept로 구현

```
[Shift+Tab → Auto-Accept 모드]

You: /angular-02-implement

Claude: [TDD-First]
        1. SocialAuthStore 테스트 작성
        2. SocialAuthStore 구현 → 테스트 통과
        3. SocialLoginPage extends IonBasePage
        4. AccountLinkingService
        5. Module 등록
        6. ng build | lint | test 통과
        → 다음: /angular-03-review
```

대규모면 `--team` 추가:
```
You: /angular-02-implement --team

Claude: [에이전트 4개 병렬]
        Store Agent → stores/
        Page Agent → pages/
        Service Agent → services/
        Component Agent → components/
```

### Step 3: 리뷰

```
You: /angular-03-review

Claude: [fork 컨텍스트에서 리뷰]

        Review: 소셜 로그인 - Score: 8/10

        [Critical] C1: SocialAuthStore에 message 타입 누락
        [Important] I1: GoogleAuthService에서 에러 타입이 any
        [Important] I2: SocialLoginPage에 destroyed$ takeUntil 누락

        → /angular-04-fix
```

### Step 4: 수정

```
You: /angular-04-fix

Claude: - [x] C1: message 타입 추가 (SocialAuthStoreMessage)
        - [x] I1: any → SocialAuthError
        - [x] I2: destroyed$ takeUntil 추가
        - [x] 회귀 테스트 추가
        - [x] Doc Sync: CHANGELOG 업데이트
        ng build | lint | test 통과

        Score: 8/10 → 10/10
```

### Step 5: 배포

```
You: /ship

Claude: 1. 테스트 통과 (62 tests)
        2. 린트 통과
        3. Doc Sync 확인
        4. git push origin feat/social-login

        ## What
        인증 모듈에 Google/Apple 소셜 로그인 추가

        ## What's NOT in This PR
        - Facebook 로그인 (V2)

        PR 올릴까요?

You: 올려.
```

---

## 3. 버그 수정 (S 사이즈)

전체 파이프라인 필요 없음. 3단계로 끝.

```
You: /debug
     결제 완료 후 스피너가 안 사라져.

Claude: [에러 분석]
        PaymentStore.finalize() 에서 isFetching = false 누락
        payment.store.ts:87

        → /{fw}-04-fix
```

```
You: /angular-04-fix

Claude: - [x] isFetching = false 추가 (payment.store.ts:87)
        - [x] 회귀 테스트: "should set isFetching to false after finalize"
        ng build | lint | test 통과
```

```
You: /ship

Claude: git push → PR 설명 생성
```

---

## 4. "이거 어떻게 만들지?" (아이디어 단계)

코드 전에 생각을 정리하고 싶을 때.

```
You: /office-hours
     팀 대시보드에 실시간 알림 기능 넣고 싶어.

Claude: 지금은 어떻게 알림 받고 있어요?

You: 이메일인데 늦어.

Claude: 말한 건 "실시간 알림"인데, 필요한 건
        "중요한 이벤트를 즉시 인지하는 것"이에요.

        V1: WebSocket + 브라우저 Notification API
        V2: 모바일 push + 알림 설정
        → /plan-ceo-review 로 스코프 확정
```

```
You: /plan-ceo-review

Claude: REDUCTION 모드.
        최소 버전: WebSocket + 토스트 알림 3종 (에러, 배포, 멘션)
        안 하는 것: 알림 히스토리, 이메일 연동, 모바일
        → /{fw}-01-plan
```

---

## 5. 디자인 먼저 정하고 싶을 때

```
You: /design-consultation
     대시보드 알림 센터 UI.

Claude: [/browse로 Slack, Discord, Linear 알림 패널 조사]
        A: "Sidebar Panel" — 사이드에서 슬라이드 인
        B: "Dropdown" — 벨 아이콘 클릭 시 드롭다운
        C: "Full Page" — 별도 알림 페이지

You: B.

Claude: DESIGN.md 생성
        - 벨 아이콘 + 뱃지 카운트
        - 드롭다운 최대 5개 표시
        - "모두 보기" 링크
        → /{fw}-01-plan에서 DESIGN.md 참조
```

---

## 6. 보안 점검

```
You: /cso
     이번 PR만.

Claude: [OWASP + STRIDE 분석]

        [Critical] auth.service.ts:45 — 토큰을 localStorage에 평문 저장
        [Critical] payment.controller.ts:23 — SQL injection 가능
        [Important] cors.config.ts:12 — origin이 * 로 설정됨

        → /{fw}-04-fix로 수정
```

---

## 7. 주간 회고 (금요일)

```
You: /retro

Claude: 이번 주 요약:
        Commits: 28 | +1,847/-592
        feat: 10, fix: 6, test: 4, chore: 8

        가장 많이 수정된 파일:
        social-auth.store.ts — 6회 (초기 설계 부족)

        Actions:
        1. Store는 message type부터 설계하기
        2. 다음 주 test ratio 50% → 60%
```

```
You: /insights

Claude: 반복 패턴 발견:
        - ComponentStore 3번 새로 만듦 → 스캐폴딩 스킬 추가?
        - S 사이즈에서 /explore 실행 2번 → task-sizing 따르기
        - CLAUDE.md 제안: "Store 설계 시 Message type 먼저"
```

---

## 8. PR 리뷰

```
You: /review-pr 456

Claude: [PR #456 분석]
        feat: 알림 센터 UI 추가
        변경 범위: 8개 파일 (+342/-12)
        리스크: Medium

        [Critical] notification.service.ts:34 — WebSocket 연결 해제 처리 없음
        [Important] NotificationList.tsx:89 — key prop에 index 사용
        [Nice-to-have] 타입명 NotifData → NotificationData

        결론: Request Changes (Critical 1건)
```

---

## 9. 의존성 관리

```
You: /dependencies

Claude: [현황 파악]
        outdated: 12개 (patch 8, minor 3, major 1)
        audit: high 1건 (lodash prototype pollution)

        Security Fix:
        - lodash 4.17.20 → 4.17.21 (high severity)

        Safe Updates (patch+minor):
        | Package | Before | After | Type |
        | rxjs | 7.8.0 | 7.8.1 | patch |
        | @angular/core | 18.1.0 | 18.2.0 | minor |
        ...

        Major (별도 PR 권장):
        - typescript 5.3 → 5.7 (breaking: decorator metadata)

        lint PASS | test PASS | build PASS
```

---

## 10. 여러 기능 동시 개발

```bash
# 터미널 1
git worktree add ../project-social feature/social-login
cd ../project-social && claude
# → /angular-01-plan부터 시작

# 터미널 2 (동시)
git worktree add ../project-notify feature/notifications
cd ../project-notify && claude
# → /react-01-plan부터 시작

# 백그라운드 테스트
claude -p "Run full test suite and fix failures" &
# → 완료되면 macOS 알림
```

---

## 자주 쓰는 패턴

### "이 코드 뭐하는 건지 모르겠어"
```
You: /explore
     payments/ 디렉토리 전체 분석해줘.
```

### "테스트만 추가하고 싶어"
```
You: /angular-05-test
     payment.store.ts 테스트 보강해줘.
```

### "리뷰만 따로 하고 싶어"
```
You: /angular-03-review
     이번 커밋 리뷰해줘.
```

### "타입 안전성만 체크"
```
You: tak-typescript-reviewer로 이번 PR 타입 체크해줘.
```

### "너무 복잡한 거 아닌지 확인"
```
You: code-simplicity-reviewer로 이 코드 검토해줘.
```

### "라이브러리 사용법 모르겠어"
```
You: best-practices-researcher로 TanStack Query v5 migration 방법 찾아줘.
```

### "남의 PR 리뷰해야 해"
```
You: /review-pr 123
```

### "패키지 업데이트 해야 하는데"
```
You: /dependencies
```

---

## 키보드 단축키

| 단축키 | 동작 |
|--------|------|
| **Shift+Tab** | Plan Mode ↔ Auto-Accept 전환 (plan 스킬 사용 시 불필요) |
| **Ctrl+O** | hook 출력 펼치기 |
| **Esc** | 현재 응답 중단 |

---

## 트러블슈팅

### "Claude가 CLAUDE.md 규칙을 무시해"
```
/compact
```
컨텍스트 정리 + 규칙 리마인더 자동 재주입

### "세션이 너무 길어져서 느려"
기능당 세션 분리. 새 터미널에서 `claude` 실행.

### "에이전트가 MCP를 못 찾아"
Context7 MCP 미설치 시 → WebSearch/WebFetch로 자동 fallback
Puppeteer MCP 미설치 시 → Playwright MCP 또는 스크린샷 요청

### "hook 에러가 뜨는데?"
```
You: hook 에러 메시지 복붙

Claude: [원인 분석 + 수정]
```
또는 `settings.json`에서 해당 hook 확인
