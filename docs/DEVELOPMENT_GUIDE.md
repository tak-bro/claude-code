# Claude Code 개발 환경 가이드
> AI-Native 개발 파이프라인 — 30개 스킬, 13개 에이전트, 자동화 훅 기반

---

## 핵심 철학: "AI와 협업하는 개발"

### 왜 이렇게 구성했나?

Claude Code는 강력하지만, **아무 규칙 없이 쓰면 컨텍스트가 오염되고 품질이 들쭉날쭉**하다.
이 설정은 3가지 문제를 해결한다:

1. **"Claude가 코드를 안 읽고 계획부터 세운다"** → Explore First 원칙
2. **"긴 세션에서 규칙을 까먹는다"** → hooks + rules 자동 주입
3. **"같은 실수를 반복한다"** → 에러 로깅 + 주간 회고 루프

### 3계층 제어 모델

```
settings.json deny (하드블록)     ← git push main 원천 차단
        ↓
hooks exit 2 (조건부 차단)        ← 보호 브랜치 파일 수정 차단
        ↓
CLAUDE.md + rules/ (가이드)       ← 컨벤션, 워크플로우 안내
```

CLAUDE.md에 "절대 하지마"라고 써도 Claude는 까먹을 수 있다.
**진짜 못하게 하려면 hooks에서 `exit 2`로 차단**해야 한다.

### 7가지 개발 원칙

| # | 원칙 | 적용 |
|---|------|------|
| 1 | **Explore Before Plan** | 코드를 안 읽고 계획 세우면 환각 발생. `/explore`로 먼저 읽기 |
| 2 | **Plan Mode에서 충분히 반복** | Shift+Tab으로 Plan Mode → 2-3번 반복 → `think hard`/`ultrathink` |
| 3 | **서브에이전트로 컨텍스트 분리** | 대규모 탐색은 서브에이전트에 위임. 메인 컨텍스트 보호 |
| 4 | **Review는 다른 컨텍스트에서** | `context: fork`로 분리. 같은 모델이라도 분리 = 더 나은 결과 |
| 5 | **결정적 제어는 hooks로** | "하지마" 대신 `exit 2`로 물리적 차단 |
| 6 | **완료 전 자가 검증** | Stop hook이 매번 체크리스트 표시. 테스트/디버그코드/컨벤션 |
| 7 | **문서를 코드와 함께 업데이트** | Doc Sync: README, CLAUDE.md, CHANGELOG 드리프트 체크 |

---

## 전체 구조

```
~/.claude/
├── CLAUDE.md                          ← 글로벌 룰 (200줄 이내)
├── settings.json                      ← hooks + deny + plugins
├── rules/                             ← 매 세션 자동 로드
│   ├── claude-philosophy.md           ← 7가지 핵심 원칙
│   ├── context-management.md          ← 세션 분리, compact 타이밍
│   ├── task-sizing.md                 ← S/M/L 스킵 가이드
│   └── thinking-guide.md             ← think ~ ultrathink 매핑
├── skills/                            ← 30개 스킬 (슬래시 커맨드)
│   ├── _shared/                       ← 공통 템플릿 (4개)
│   ├── plan/                          ← base plan
│   ├── {fw}-{01~05}-{phase}/         ← 프레임워크별 5단계 x 3
│   └── (10개 확장 스킬)
├── agents/                            ← 13개 전문 에이전트
└── docs/                              ← 가이드 문서
```

---

## 워크플로우

### 전체 흐름

```
 아이디어          온보딩           기능 개발                    배포      회고
────────── ──────────── ──────────────────────────── ────── ──────
/office    /generate    /explore → /{fw}-01-plan     /ship  /retro
-hours     -codebase     → /{fw}-02-implement                → /insights
           -context      → /{fw}-03-review
/plan-ceo              → /{fw}-04-fix
-review                → /{fw}-05-test → /qa
```

### 작업 규모별 스킵

| 규모 | 기준 | 워크플로우 |
|------|------|-----------|
| **S** | 1-2 파일 | `/debug` → `/{fw}-04-fix` → `/ship` |
| **M** | 3-10 파일 | `/explore` → `01-plan` → `02-implement` → `03-review` → `/ship` |
| **L** | 10+ 파일 | 전체 파이프라인 + `--team` 모드 (에이전트 병렬) |

---

## 스킬 카탈로그 (30개)

### 기본 파이프라인 (18개)

**Plan → Implement → Review → Fix → Test** 의 5단계를 Angular, React, TypeScript 3개 프레임워크에 적용.

| 단계 | 역할 | 핵심 동작 |
|------|------|----------|
| `/{fw}-01-plan` | 계획 수립 | Phase 0 Explore + Phase 0.5 Rethink + "What's NOT in This PR" |
| `/{fw}-02-implement` | TDD 구현 | 실패 테스트 먼저 → 최소 코드 → `--team`으로 병렬 가능 |
| `/{fw}-03-review` | 코드 리뷰 | `context: fork` 분리 실행. 10점 스코어링 |
| `/{fw}-04-fix` | 이슈 수정 | references/common-fixes.md의 Before/After 패턴 참조 |
| `/{fw}-05-test` | 테스트 보강 | 행위 기반 네이밍 + AAA 패턴 |

공통 스킬:
| 스킬 | 역할 |
|------|------|
| `/plan` | 모든 fw plan이 참조하는 베이스 |
| `/debug` | 증상→원인→수정 디버깅 |
| `/generate-codebase-context` | `.claude/llms.txt` 생성. 온보딩 첫 단계 |

### 확장 스킬 (10개)

| 스킬 | 역할 | 언제 쓰나 |
|------|------|----------|
| `/explore` | 읽기 전용 코드 분석 | plan 전, 모듈 파악할 때 |
| `/plan-ceo-review` | 10-star 제품 사고 | 기능 스코프 재정의할 때 |
| `/office-hours` | 아이디어 정제 | 코드 전, 니즈 파악할 때 |
| `/design-consultation` | UI/UX 리서치 → DESIGN.md | 디자인 결정 필요할 때 |
| `/qa` | diff → 브라우저 테스트 | 웹 프로젝트 배포 전 |
| `/browse` | Playwright 브라우저 자동화 | 수동 UI 테스트 대체 |
| `/ship` | 테스트→린트→push→PR | 배포 준비 완료 시 |
| `/cso` | OWASP + STRIDE 보안 감사 | 보안 민감 코드 작성 후 |
| `/retro` | git log 기반 주간 회고 | 매주 금요일 |
| `/insights` | 워크플로우 개선 분석 | retro 후, 반복 패턴 발견 시 |
| `/review-pr` | GitHub PR 코드 리뷰 | 남의 PR 리뷰할 때 |
| `/dependencies` | 의존성 점검 + 업데이트 | outdated/audit 관리 |

---

## 자동화: Hooks

사람이 기억하는 건 한계가 있다. **hooks가 대신 기억한다.**

| 트리거 | 동작 | 왜? |
|--------|------|-----|
| `git push main/develop` | 차단 (`exit 2`) | feature 브랜치 강제 |
| `git push -f` | 차단 | 히스토리 보호 |
| main에서 파일 수정 | 차단 | 실수 방지 |
| `.ts` 파일 저장 | 자동 prettier | 일관된 포맷 |
| 세션 시작 | git log + status 주입 | 컨텍스트 즉시 파악 |
| compact/resume 후 | 규칙 리마인더 재주입 | 규칙 유실 방지 |
| Claude 응답 완료 | 자가 검증 프롬프트 | 테스트/디버그코드/컨벤션 체크 |
| 작업 완료 | macOS 알림 | 백그라운드 작업 인지 |

---

## 에이전트 (13개)

스킬이 **프로세스**(어떤 순서로 할지)라면, 에이전트는 **전문성**(누가 할지)이다.

| 에이전트 | 전문 영역 |
|---------|----------|
| `angular-componentstore-expert` | ComponentStore, Module-based, Ionic 패턴 |
| `react-feature-library-expert` | Feature Library, TanStack Query, Zustand, Nx |
| `tak-typescript-expert` | tak 컨벤션 TypeScript 구현 |
| `tak-typescript-reviewer` | 타입 안전성, export 규칙, 컨벤션 준수 |
| `code-simplicity-reviewer` | YAGNI, 복잡도 감소, 불필요한 추상화 제거 |
| `api-designer` | REST/GraphQL 설계, TypeScript 타입 생성 |
| `performance-analyzer` | 렌더링, 번들, 메모리 최적화 |
| `test-strategy-expert` | 유닛/통합/E2E 테스트 전략 |
| `codebase-researcher` | 아키텍처 분석, 패턴 추출 |
| `best-practices-researcher` | 기술 베스트 프랙티스 + 프레임워크 문서 리서치 |
| `design-implementation-reviewer` | Figma 디자인 검증 + UI 구현 (리뷰/코딩 2모드) |
| `git-workflow-expert` | 커밋, PR, 브랜치 전략 |
| `tak-document-writer` | 문서 스타일 리뷰 (tak 스타일 가이드) |

---

## Thinking 가이드

Claude에게 생각의 깊이를 지시할 수 있다.

| 키워드 | 용도 | 예시 |
|--------|------|------|
| `think` | 일반 구현 | 함수 하나 작성, 타입 정의 |
| `think hard` | API/상태 설계 | Store 구조, 모듈 경계, 데이터 흐름 |
| `think harder` | 마이그레이션 | 라이브러리 교체, 대규모 리팩토링 |
| `ultrathink` | 아키텍처 결정 | 기술 스택 선택, 트레이드오프 분석 |

---

## 컨텍스트 관리

| 규칙 | 이유 |
|------|------|
| 기능당 세션 분리 | 여러 기능 섞으면 컨텍스트 오염 |
| Claude가 규칙 무시하면 `/compact` | 컨텍스트 정리 + 리마인더 재주입 |
| 대규모 탐색은 서브에이전트로 | 메인 컨텍스트 보호 |
| `context: fork` 스킬 활용 | 리뷰/보안감사는 분리 컨텍스트가 더 정확 |

---

## 병렬 개발 (Worktree)

```bash
# 터미널 1: 환불 기능
git worktree add ../project-refund feature/refund
cd ../project-refund && claude

# 터미널 2: WebSocket (동시)
git worktree add ../project-ws feature/websocket
cd ../project-ws && claude

# 백그라운드 작업
claude -p "Run full test suite and fix failures" &
# → Notification hook이 완료 알림
```

---

## 빠른 참조

| 하고 싶은 것 | 명령 |
|-------------|------|
| 처음 보는 프로젝트 | `/generate-codebase-context` → `/explore` |
| Angular 새 기능 | `/explore` → `/angular-01-plan` → `02-implement` → `03-review` → `04-fix` → `05-test` → `/ship` |
| React 새 기능 | `/explore` → `/react-01-plan` → `02-implement` → `03-review` → `04-fix` → `05-test` → `/ship` |
| TypeScript 유틸리티 | `/typescript-01-plan` → `02-implement` → `03-review` → `/ship` |
| 버그 수정 (S) | `/debug` → `/{fw}-04-fix` → `/ship` |
| 새 제품 아이디어 | `/office-hours` → `/plan-ceo-review` → `/{fw}-01-plan` |
| 디자인 결정 | `/design-consultation` |
| 보안 점검 | `/cso` |
| 브라우저 QA | `/qa` |
| 주간 회고 | `/retro` → `/insights` |

---

## 수치로 보는 설정

| 항목 | 수량 |
|------|------|
| 스킬 (slash commands) | 30개 |
| 전문 에이전트 | 13개 |
| 자동화 훅 | 8개 (PreToolUse 2, PostToolUse 1, SessionStart 2, Stop 1, Notification 1) |
| deny 룰 | 13개 (destructive 5 + credential 8) |
| 자동 로드 규칙 (rules/) | 4개 |
| 공유 리소스 (_shared/) | 4개 |
| context: fork 스킬 | 7개 (explore, 3 review, cso, plan-ceo-review, review-pr) |
| 지원 프레임워크 | 3개 (Angular, React, TypeScript) |
