# Claude Code Configuration

AI-Native 개발 환경 설정 — 62개 스킬 + 16개 에이전트 + 자동화 훅

## 새 PC 설치

```bash
# 1. 이 레포 클론
git clone git@github.com:tak-bro/claude-code.git ~/.claude

# 2. 플러그인 설치 (Claude Code CLI에서 실행)
/install-plugin thedotmack/claude-mem
/install-plugin claude-plugins-official/frontend-design
/install-plugin claude-plugins-official/code-simplifier
/install-plugin claude-plugins-official/code-review
/install-plugin claude-plugins-official/swift-lsp

# 3. 마켓플레이스 플러그인 설치
/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
```

MCP 서버 설정은 `settings.json`에 포함되어 클론 시 자동 적용.

## 구조

```
~/.claude/
├── CLAUDE.md                    ← 글로벌 룰
├── settings.json                ← hooks, deny rules, plugins
├── rules/                       ← 매 세션 자동 로드
├── skills/                      ← 62개 스킬
│   ├── _shared/                 ← 공통 템플릿
│   ├── {fw}-{01~05}-{phase}/   ← 프레임워크별 5단계 x 6
│   └── 32개 범용 스킬
├── agents/                      ← 16개 전문 에이전트
└── docs/                        ← 가이드 문서
```

## 워크플로우

| 규모 | 흐름 |
|------|------|
| **S** (1-2 파일) | `/investigate` → `/{fw}-04-fix` → `/ship` |
| **M** (3-10 파일) | `/explore` → `/{fw}-01-plan` → `/plan-ceo-review` → `/plan-eng-review` → `02-implement` → `/simplify` → `03-review` → `/ship` |
| **L** (10+ 파일) | `/explore` → `/{fw}-01-plan` → `/plan-ceo-review` → `/plan-eng-review` → `02-implement` → `/batch` → `03-review` → `/qa` → `/ship` |

## 스킬 (62개)

### 프레임워크 파이프라인 (30개)

`/{fw}-01-plan` → `02-implement` → `03-review` → `04-fix` → `05-test`

지원: Angular, React, TypeScript, Kotlin, Swift, Node.js

### 공통 스킬 (3개)

| 스킬 | 설명 |
|------|------|
| `/plan` | 모든 프레임워크 plan이 참조하는 베이스 계획 스킬 |
| `/investigate` | 근본 원인 디버깅. 4단계: investigate → analyze → hypothesize → implement |
| `/generate-codebase-context` | `.claude/llms.txt` 생성. 프로젝트 온보딩 첫 단계 |

### 아이디어/계획 (4개)

| 스킬 | 설명 |
|------|------|
| `/office-hours` | 아이디어 정제. 코드 전 니즈 파악 |
| `/plan-ceo-review` | 10-star 제품 관점. 기능 스코프 재정의 |
| `/plan-eng-review` | 엔지니어링 아키텍처 리뷰. 설계 검증 |
| `/plan-design-review` | 디자이너 관점 계획 리뷰. 0-10 스코어링 |

### 코드 리뷰/QA (5개)

| 스킬 | 설명 |
|------|------|
| `/review` | 프리랜딩 PR 코드 리뷰. fix-first 접근 |
| `/review-pr` | GitHub PR 코드 리뷰. 남의 PR 리뷰 |
| `/qa` | 영향도 분석 + 정적 검증 + 테스트 + 브라우저 검증 + 자동 수정 |
| `/qa-only` | 리포트 전용 QA. 수정 없이 버그 리포트만 |
| `/design-review` | 라이브 사이트 디자인 감사 + 수정 |

### 구현 지원 (4개)

| 스킬 | 설명 |
|------|------|
| `/explore` | 읽기 전용 코드 분석. plan 전 모듈 파악 |
| `/batch` | 격리된 work tree에서 병렬 실행 |
| `/simplify` | 코드 단순화 + 품질 체크 (플러그인) |
| `/find-skills` | 스킬 검색 + 설치 도우미 |

### 보안 (1개)

| 스킬 | 설명 |
|------|------|
| `/security-scan` | 7-앵글 보안 취약점 스캔 |

### 배포/문서/회고 (3개)

| 스킬 | 설명 |
|------|------|
| `/ship` | 테스트 → 린트 → push → PR 설명 생성 |
| `/document-release` | 배포 후 문서 동기화 |
| `/retro` | git log 기반 주간 회고 |

### 품질/성능 (3개)

| 스킬 | 설명 |
|------|------|
| `/health` | 코드 품질 대시보드. 0-10 점수 |
| `/benchmark` | 성능 회귀 탐지 |
| `/tech-debt` | 세션 중 생긴 부채 정리 |

### 디자인 (6개)

| 스킬 | 설명 | 예제 |
|------|------|------|
| `/gpt-taste` | Awwwards급 프리미엄 디자인 + GSAP 모션. 랜딩 페이지, 마케팅, 포트폴리오 | `/gpt-taste` 랜딩 페이지 만들어줘 |
| `/design-taste-frontend` | React/Next.js 프론트엔드 엔지니어링. 대시보드, SaaS, 앱 UI | `/design-taste-frontend` 어드민 대시보드 리디자인해줘 |
| `/redesign-existing-projects` | 기존 사이트 프리미엄 업그레이드. AI 슬롭 패턴 제거 | `/redesign-existing-projects` src/pages/ 전체 UI 멋지게 개선해줘 |
| `/minimalist-ui` | 미니멀 에디토리얼 스타일. 따뜻한 모노크롬, 타이포 대비, 벤토 그리드 | `/minimalist-ui` 블로그 레이아웃 만들어줘 |
| `/industrial-brutalist-ui` | 스위스 타이포 + 밀리터리 터미널 미학. 데이터 대시보드, 포트폴리오용 | `/industrial-brutalist-ui` 모니터링 대시보드 만들어줘 |
| `/full-output-enforcement` | LLM 코드 잘림 방지. 완전한 코드 생성 강제 | 다른 디자인 스킬과 함께 사용 |

> **gpt-taste vs design-taste-frontend:** gpt-taste는 디자인 감각 (비주얼 임팩트, 스크롤 애니메이션, AIDA 구조), design-taste-frontend는 프론트엔드 엔지니어링 (컴포넌트 아키텍처, CSS 하드웨어 가속, 접근성). 같이 쓸 수 있음: gpt-taste로 방향 잡고 → design-taste-frontend로 구현 품질 다듬기.
>
> **design-taste-frontend vs frontend-design (플러그인):** design-taste-frontend는 React/Next.js 전용, 구체적 수치/금지 규칙 (Inter 금지, 보라색 금지, Tailwind 필수 등 20K+ 규칙). frontend-design 플러그인은 스택 무관 범용 가이드 (4K, 방향성만 제시). React/Next.js면 design-taste-frontend, Vue/바닐라 등 기타 스택이면 frontend-design 플러그인.

**디자인 워크플로우:**

| 목적 | 플로우 |
|------|--------|
| 기존 UI 리디자인 | `/explore` → `/redesign-existing-projects` → `/design-review` → `/qa` → `/ship` |
| 새 페이지 제작 | `/gpt-taste` (or `/design-taste-frontend`) → `/design-review` → `/qa` → `/ship` |
| 스타일 프리셋 적용 | `/minimalist-ui` 또는 `/industrial-brutalist-ui`를 제작 스킬과 함께 사용 |

> **gpt-taste vs design-taste-frontend:** gpt-taste는 디자인 감각 (비주얼 임팩트, 스크롤 애니메이션, AIDA 구조), design-taste-frontend는 프론트엔드 엔지니어링 (컴포넌트 아키텍처, CSS 하드웨어 가속, 접근성). 같이 쓸 수 있음: gpt-taste로 방향 잡고 → design-taste-frontend로 구현 품질 다듬기.
>
> **design-taste-frontend vs frontend-design (플러그인):** design-taste-frontend는 React/Next.js 전용, 구체적 수치/금지 규칙 (Inter 금지, 보라색 금지, Tailwind 필수 등 20K+ 규칙). frontend-design 플러그인은 스택 무관 범용 가이드 (4K, 방향성만 제시). React/Next.js면 design-taste-frontend, Vue/바닐라 등 기타 스택이면 frontend-design 플러그인.
>
> **taste-skill vs ui-ux-pro-max (플러그인):** taste-skill은 디자인 "감각" (슬롭 금지, GSAP 모션, 레이아웃 랜덤화). ui-ux-pro-max는 디자인 "규칙" (접근성 4.5:1 대비, 44x44px 터치, 99 UX 가이드라인, 161 팔레트, 57 폰트 조합). taste-skill이 "예쁘게", ui-ux-pro-max가 "제대로". 같이 쓰면 감각 + 규칙 모두 커버.

**디자인 워크플로우:**

| 목적 | 플로우 |
|------|--------|
| 기존 UI 리디자인 | `/explore` → `/redesign-existing-projects` → `/design-review` → `/qa` → `/ship` |
| 새 페이지 제작 | `/gpt-taste` (or `/design-taste-frontend`) → `/design-review` → `/qa` → `/ship` |
| 스타일 프리셋 적용 | `/minimalist-ui` 또는 `/industrial-brutalist-ui`를 제작 스킬과 함께 사용 |

### 유틸리티 (3개)

| 스킬 | 설명 |
|------|------|
| `/learn` | 프로젝트 학습 관리. 세션 간 학습 조회/정리 |
| `/context-save` / `/context-restore` | 작업 컨텍스트 저장/복원 |
| `/make-pdf` | 마크다운 → PDF 변환 |

## 에이전트 (16개)

| 에이전트 | 전문 영역 |
|---------|----------|
| `angular-componentstore-expert` | ComponentStore, Module-based, Ionic |
| `react-feature-library-expert` | Feature Library, TanStack Query, Zustand, Nx |
| `kotlin-android-expert` | Jetpack Compose, Hilt, Room |
| `swift-ios-expert` | SwiftUI, Combine, SPM |
| `nodejs-backend-expert` | Express, NestJS, Prisma |
| `tak-typescript-expert` | tak 컨벤션 TypeScript |
| `tak-typescript-reviewer` | 타입 안전성, export 규칙 |
| `code-simplicity-reviewer` | YAGNI, 복잡도 감소 |
| `api-designer` | REST/GraphQL 설계 |
| `performance-analyzer` | 렌더링, 번들, 메모리 |
| `test-strategy-expert` | 유닛/통합/E2E 전략 |
| `codebase-researcher` | 아키텍처 분석, 패턴 추출 |
| `best-practices-researcher` | 기술 베스트 프랙티스 리서치 |
| `design-implementation-reviewer` | Figma 디자인 검증 + UI 구현 |
| `git-workflow-expert` | 커밋, PR, 브랜치 전략 |
| `tak-document-writer` | 문서 스타일 리뷰 |

## 플러그인

- **claude-mem** — 크로스 세션 메모리 (smart-explore, mem-search, timeline)
- **frontend-design** — 프로덕션 그레이드 UI 생성
- **code-simplifier** — 코드 단순화 리뷰
- **code-review** — PR 코드 리뷰
- **swift-lsp** — Swift LSP 통합
- **ui-ux-pro-max** — UI/UX 디자인 인텔리전스. 50+ 스타일, 161 팔레트, 57 폰트 조합, 99 UX 가이드라인, 접근성 규칙

## 참고

- [docs/USAGE_GUIDE.md](docs/USAGE_GUIDE.md) — 시나리오별 사용 예시
- [docs/DEVELOPMENT_GUIDE.md](docs/DEVELOPMENT_GUIDE.md) — 전체 개발 환경 상세 가이드
- 일부 스킬은 [garrytan/gstack](https://github.com/garrytan/gstack)에서 핵심 워크플로우 추출
- 디자인 스킬은 [leonxlnx/taste-skill](https://github.com/leonxlnx/taste-skill)에서 설치
