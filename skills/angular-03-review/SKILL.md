---
name: angular-03-review
model: sonnet
effort: medium
description: "Angular code review. Triggers on 'angular review', '앵귤러 리뷰', 'angular code review', 'ng review', 'angular 검토'."
context: fork
tools: Read, Bash, Glob, Grep
---

# Angular Review

Review Angular code and generate Fix Checklist for next phase.

**Agents:** angular-componentstore-expert, tak-typescript-reviewer, code-simplicity-reviewer

**Context: fork** — Review in a separate context from implementation (Test-Time Compute: same model but separated context = better results).

**Smart Review Routing:** Backend code → code-reviewer, UI code → design-reviewer, security-related → `/cso`

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

### 🚫 Never Do
- Skip Critical checks
- Give vague feedback without fix examples
- Approve code with Critical issues

---

## Checks (Priority Order)

### [Critical] (Auto-Fail)
1. Standalone component used (must be Module-based)?
2. Component -> API direct call (bypassing ComponentStore)?
3. `export default` found?
4. Business logic in presentational component?
5. Missing message pattern in ComponentStore state?
6. (Ionic) Direct controller usage without service wrapper?
7. (Ionic) Page not extending IonBasePage?
8. (Ionic) Missing super.ionViewWillEnter() call?

### [Important]
9. Missing component-level `destroyed$` cleanup?
10. Page vs Component separation violated?
11. Store not provided in Module's providers?
12. Guards order wrong? (App -> Auth -> Feature)
13. API methods missing `$` suffix?
14. (Ionic) IonicRouteStrategy not configured?
15. (Multi-env) LocalStorage keys missing `_${env}` suffix?
16. (TanStack Query) Missing query keys factory pattern?

### [Nice-to-have]
17. Could selectors be combined?
18. Effect error handling could be better?
19. Naming could be clearer?

---

## Decision Tree

```
Standalone component? → [Critical] Convert to Module-based
Component → API direct? → [Critical] Use ComponentStore
export default? → [Critical] Convert to named export (except App/config entrypoint)
Business logic in Component? → [Critical] Move to Page or Store
Missing message pattern? → [Critical] Add message field to state
(Ionic) Direct controller? → [Critical] Use service wrapper
(Ionic) Not extending IonBasePage? → [Critical] Add extends IonBasePage
(Ionic) Missing super.ionViewWillEnter()? → [Critical] Add super call
Missing destroyed$ cleanup? → [Important] Add cleanup
Store not in Module providers? → [Important] Add to providers
Wrong Guard order? → [Important] Fix to App → Auth → Feature
(Ionic) Missing IonicRouteStrategy? → [Important] Configure in app.module
(Multi-env) Missing _${env}? → [Important] Add suffix
(TanStack Query) No query keys? → [Important] Add factory pattern
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
| Angular Expert | ComponentStore patterns, Module arch | angular-componentstore-expert |
| Code Quality | TypeScript, exports, naming | tak-typescript-reviewer |
| Simplicity | Over-engineering, complexity | code-simplicity-reviewer |

---

## Output
(see `_shared/output-templates.md` — Review Output)

→ If issues exist: `/angular-04-fix`, otherwise: `/qa` or `/ship`
