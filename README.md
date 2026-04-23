# Claude Code Configuration

AI-Native 개발 환경 설정 — 56개 스킬 + 16개 에이전트 + 자동화 훅

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
```

MCP 서버 설정은 `settings.json`에 포함되어 클론 시 자동 적용.

## 구조

```
~/.claude/
├── CLAUDE.md                    ← 글로벌 룰
├── settings.json                ← hooks, deny rules, plugins
├── rules/                       ← 매 세션 자동 로드
├── skills/                      ← 56개 스킬
│   ├── _shared/                 ← 공통 템플릿
│   ├── {fw}-{01~05}-{phase}/   ← 프레임워크별 5단계 x 6
│   └── 28개 범용 스킬
├── agents/                      ← 16개 전문 에이전트
└── docs/                        ← 가이드 문서
```

## 워크플로우

| 규모 | 흐름 |
|------|------|
| **S** (1-2 파일) | `/investigate` → `/{fw}-04-fix` → `/ship` |
| **M** (3-10 파일) | `/explore` → `/{fw}-01-plan` → `02-implement` → `/simplify` → `03-review` → `/ship` |
| **L** (10+ 파일) | 전체 파이프라인 + `/batch` |

## 스킬 (58개)

### 프레임워크 파이프라인 (30개)

`/{fw}-01-plan` → `02-implement` → `03-review` → `04-fix` → `05-test`

지원: Angular, React, TypeScript, Kotlin, Swift, Node.js

### 범용 스킬 (28개)

| 카테고리 | 스킬 |
|---------|------|
| 아이디어/계획 | `/office-hours`, `/plan-ceo-review`, `/plan-eng-review`, `/plan-design-review` |
| 코드 리뷰/QA | `/review`, `/review-pr`, `/qa`, `/qa-only`, `/design-review` |
| 디버깅/보안 | `/investigate`, `/investigate`, `/security-scan` |
| 디자인 | `/design-consultation` |
| 구현 지원 | `/explore`, `/plan`, `/batch`, `/simplify` |
| 배포/문서 | `/ship`, `/document-release`, `/retro` |
| 품질/성능 | `/health`, `/benchmark`, `/tech-debt` |
| 유틸리티 | `/learn`, `/context-save`, `/context-restore`, `/make-pdf`, `/generate-codebase-context` |

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

## 참고

- [docs/USAGE_GUIDE.md](docs/USAGE_GUIDE.md) — 시나리오별 사용 예시
- [docs/DEVELOPMENT_GUIDE.md](docs/DEVELOPMENT_GUIDE.md) — 전체 개발 환경 상세 가이드
- 14개 스킬은 [garrytan/gstack](https://github.com/garrytan/gstack)에서 핵심 워크플로우만 추출
