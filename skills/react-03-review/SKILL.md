---
name: react-03-review
description: "React 코드 리뷰. Triggers on 'react review', '리액트 리뷰', 'react code review', 'react 검토'."

tools: Read, Bash, Glob, Grep
---

# React Review

Review React code and generate Fix Checklist for next phase.

## Scope
Review ONLY changed files. Run `git diff --name-only` first to identify targets.
Do NOT scan or review the entire codebase.
**Agents:** react-feature-library-expert, tak-typescript-reviewer, code-simplicity-reviewer, design-implementation-reviewer (if UI)


**Smart Review Routing:** UI → design-implementation-reviewer, security → `/security-scan`

---

## Pre-Review

**From plan and implement outputs:**
1. Review Focus items from plan
2. Implementation notes from implement
3. Files created/modified list

---

## Boundaries

### Always Do
- Check ALL Critical items (auto-fail if missed)
- Provide specific file:line references
- Include fix code snippets
- Generate Fix Checklist for next phase

### [Warning] Ask First
- Suggest major refactoring
- Recommend architecture changes
- Creating new Zustand store

### 🚫 Never Do
- Skip Critical checks
- Give vague feedback without fix examples
- Approve code with Critical issues

---

## Checks (Priority Order)

### [Critical] (Auto-Fail)
1. `export default` found? → Must convert to named (except App/config entrypoint)
2. Missing barrel exports (index.ts)?
3. Component → API direct call (bypassing hooks)?
4. `any` without justification?
5. Business logic in presentational component?

### [Important]
6. `useState` for shared state? (should use Zustand)
7. Props > 10? (props explosion)
8. Component > 200 lines?
9. Missing error boundary for async operations?
10. TanStack Query key conflicts?
11. Relative imports across libs? (should use @{projectName}/*)
12. Missing loading/error states in data fetching?

### [Nice-to-have]
13. Could be simpler?
14. Missing cn() for Tailwind class merging?
15. Naming could be clearer?
16. Could selectors be memoized?

---

## Decision Tree

```
Default exports/imports? → [Critical] Convert to named
Barrel exports missing? → [Critical] Add index.ts
Component → API direct? → [Critical] Use TanStack Query hooks
any without comment? → [Critical] Add types or justification
Business logic in Component? → [Critical] Move to hook or Page
Relative imports across libs? → [Important] Use @{projectName}/*
useState for shared state? → [Important] Use Zustand
Component > 200 lines? → [Important] Consider splitting
Props > 10? → [Important] Props explosion - split component
Missing error boundary? → [Important] Add ErrorBoundary
Query key conflicts? → [Important] Use createQueryKeys factory
Missing loading/error UI? → [Important] Add Suspense/ErrorBoundary
All pass → Score 10/10
```

---

## Scoring
(see `_shared/scoring-table.md`)

## Verification Commands
(see `_shared/verification-commands.md`)

---

## --team Mode
(see `_shared/team-mode.md`)

### Team Composition

| Role | Focus | Agent |
|------|-------|-------|
| React Expert | Feature Library, TanStack Query, Zustand | react-feature-library-expert |
| Code Quality | TypeScript, patterns, exports | tak-typescript-reviewer |
| Simplicity | Over-engineering, complexity | code-simplicity-reviewer |
| UI/Design | Figma match, visual QA | design-implementation-reviewer |

---

## Output
(see `_shared/output-templates.md` — Review Output)

→ If issues: `/react-04-fix`, if no issues: `/qa` or `/ship`
