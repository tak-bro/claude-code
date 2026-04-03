---
name: react-02-implement
description: "React 기능 구현. Feature Library pattern (Nx monorepo). Triggers on 'react implement', '리액트 구현', 'react coding', 'react 개발'."
tools: Read, Bash, Glob, Grep, Write, Edit, Agent
---

# React Implement

Implement React features using Feature Library pattern (Nx monorepo).

**Agents:** react-feature-library-expert, design-implementation-reviewer, best-practices-researcher

**For detailed code patterns, reference `references/` directory:**
- `references/feature-library-pattern.md` — Feature Library structure and patterns

---

## Pre-Implementation
1. Plan approved?
2. Files to create/modify identified?
3. Implementation Checklist ready?

---

## TDD-First Principle (Claude Philosophy)
1. Write failing tests first → minimum code → refactoring
2. After plan approval, switch to Auto-Accept Mode with Shift+Tab
3. For large features, use `use subagents` to separate main context

---

## Boundaries

### Always Do
- Named exports only (`export const`)
- Barrel exports (index.ts) for every folder
- Component → Hook → TanStack Query → API flow
- Zustand for shared state, useState for local only

### [Warning] Ask First
- Adding new dependencies
- Modifying shared utilities

### 🚫 Never Do
- `export default` (except App entrypoint, config)
- `useState` for shared state
- Direct API calls in components
- Skip barrel exports
- `any` without justification

---

## Structure

```bash
npx nx g @nx/js:lib {feature} --directory=libs/{feature}
```

```
libs/{feature}/
├── apis/index.ts      # API functions
├── hooks/index.ts     # TanStack Query hooks
├── types/index.ts     # Types
├── consts/index.ts    # Constants
└── index.ts           # Barrel (REQUIRED)
```

**Flow:** Component → Hook → TanStack Query → API

---

## Anti-Over-Engineering (WET > DRY)
- **2 simple > 1 complex**: Prefer two simple components over mega-component
- **Intentional duplication OK**: If it preserves clarity
- **~200 lines guideline**: Consider splitting if exceeded
- **~5 business props healthy**: 10+ is a smell

---

## Checklist

- [ ] **Create lib:** `npx nx g @nx/js:lib {feature} --directory=libs/{feature}`
- [ ] Named exports ONLY
- [ ] Barrel exports ALL folders
- [ ] @{projectName}/* aliases
- [ ] cn() for Tailwind
- [ ] Component → Hook → TanStack Query → API
- [ ] Zustand (global) | TanStack Query (server) | useState (local)
- [ ] No `any` without justification comment
- [ ] ~200 lines guideline, ~5 business props
- [ ] Verification commands pass

---

## --team Mode
(see `_shared/team-mode.md`)

| Role | Responsibility | Files |
|------|---------------|-------|
| API Agent | API functions, endpoints | `apis/` |
| Hook Agent | TanStack Query hooks, Zustand | `hooks/` |
| Component Agent | React components, UI | `components/` |
| Types Agent | TypeScript types, constants | `types/`, `consts/` |

---

## Verification Commands
(see `_shared/verification-commands.md`)

## Output
(see `_shared/output-templates.md`)

→ Next: `/react-03-review`
