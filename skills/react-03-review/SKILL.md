---
name: react-03-review
description: "React 코드 리뷰. Triggers on 'react review', '리액트 리뷰', 'react code review', 'react 검토'."

tools: Read, Bash, Glob, Grep
---

# React Review

Review React code and generate Fix Checklist.

## Scope
Review ONLY changed files. Run `git diff --name-only` first to identify targets.
Do NOT scan or review the entire codebase.
**Agents:** tak-typescript-reviewer, code-simplicity-reviewer, design-implementation-reviewer (if UI)


**Smart Review Routing:** UI → design-implementation-reviewer, security → `/cso`

---

## Checks (Priority Order)

### [Critical] (Auto-Fail)
1. `export default` found? → Must convert to named (except App/config entrypoint)
2. Missing barrel exports (index.ts)?
3. Component → API direct call (bypassing hooks)?
4. `any` without justification?

### [Important]
5. `useState` for shared state? (should use Zustand)
6. Props > 10? (props explosion)
7. Component > 200 lines?

### [Nice-to-have]
8. Could be simpler?
9. Missing cn() for Tailwind?
10. Naming could be clearer?

---

## Decision Tree

```
Default exports/imports? → [Critical] Convert to named
Barrel exports missing? → [Critical] Add index.ts
Component → API direct? → [Critical] Use TanStack Query hooks
any without comment? → [Critical] Add types or justification
Relative imports across libs? → [Important] Use @{projectName}/*
useState for shared state? → [Important] Use Zustand
Component > 200 lines? → [Important] Consider splitting
Props > 10? → [Important] Props explosion - split component
All pass → Score 10/10
```

---

## Scoring
(see `_shared/scoring-table.md`)

## Verification Commands
(see `_shared/verification-commands.md`)

## --team Mode
(see `_shared/team-mode.md`)

| Role | Focus | Agent |
|------|-------|-------|
| Code Quality | TypeScript, patterns, exports | tak-typescript-reviewer |
| Simplicity | Over-engineering, complexity | code-simplicity-reviewer |
| UI/Design | Figma match, visual QA | design-implementation-reviewer |

---

## Output
(see `_shared/output-templates.md` — Review Output)

→ If issues: `/react-04-fix`, if no issues: `/qa` or `/ship`
