---
name: angular-04-fix
description: "Angular review issue fixes. Triggers on 'angular fix', '앵귤러 수정', 'angular 이슈 해결', 'ng fix'."
tools: Read, Bash, Glob, Grep, Write, Edit
model: sonnet

# Angular Fix

Fix issues found in review, prioritized by severity.

**Agents:** angular-componentstore-expert, tak-typescript-expert

**Fix pattern references:**
- `references/common-fixes.md` — Before/After code for all common Angular fixes
- `../angular-02-implement/references/componentstore-pattern.md`
- `../angular-02-implement/references/ionic-patterns.md`
- `../angular-02-implement/references/tanstack-query-pattern.md`

---

## Pre-Fix

**From review output, load:**
1. Fix Checklist (Critical → Important → Nice-to-have)
2. Specific file:line references
3. Fix code snippets provided

---

## Boundaries

### Always Do
- Fix ALL [Critical] issues (no exceptions)
- Run verification after each fix
- Mark completed items in checklist
- Keep fixes minimal and focused
- Regression testing required for all bug fixes (Claude Philosophy)

### [Warning] Ask First
- Skip [Important] issues
- Refactor beyond fix scope
- Add new features while fixing

### 🚫 Never Do
- Skip [Critical] issues
- Change logic beyond the fix
- Introduce new patterns
- Leave verification failing

---

## Fix Priority

```
[Critical] (MUST fix - no exceptions)
    │
    ├─ Standalone → Module-based
    ├─ Component → API direct → use ComponentStore
    ├─ export default → named export (except App/config entrypoint)
    ├─ Business logic in Component → move to Page/Store
    ├─ Missing message pattern → add to state
    ├─ (Ionic) Direct controller → service wrapper
    ├─ (Ionic) Not extending IonBasePage → add extends
    └─ (Ionic) Missing super.ionViewWillEnter() → add call
    │
    ▼
[Important] (SHOULD fix)
    │
    ├─ Missing destroyed$ cleanup
    ├─ Page/Component separation
    ├─ Store not in Module providers
    ├─ Wrong Guard order
    ├─ (Ionic) Missing IonicRouteStrategy → configure
    ├─ (Multi-env) Missing _${env} suffix
    └─ (TanStack Query) Missing query keys factory
    │
    ▼
[Nice-to-have] (OPTIONAL)
```

---

## Doc Sync (Claude Philosophy)
After fixing, verify:
- README drift check
- CLAUDE.md needs update?
- CHANGELOG needs update?

## Self-Verification (Stop hook integration)
Before completion, verify:
- [ ] Tests pass?
- [ ] Debug code removed?
- [ ] Conventions followed?

---

## Checklist

- [ ] ALL [Critical] issues fixed
- [ ] [Important] issues addressed (or justified skip)
- [ ] `ng build` passes (if available)
- [ ] `{pm} run lint` passes (if available)
- [ ] `ng test` passes (if available)
- [ ] `ng build --configuration=production` passes (if available)
- [ ] Fix report generated
- [ ] Re-review decision made

## Verification Commands
(see `_shared/verification-commands.md`)

## Re-Review Criteria

| Condition | Action |
|-----------|--------|
| Had Critical issues | → Run `/angular-03-review` again (required) |
| Had only Important issues | → Self-check and complete |
| Final score 8/10 or higher | → Complete |
| Final score below 8/10 | → Run `/angular-03-review` again |

---

## Output
(see `_shared/output-templates.md` — Fix Output)

→ Next: `/angular-03-review` (re-review) or `/qa` or `/ship`
