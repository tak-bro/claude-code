---
name: ts-04-fix
description: "TypeScript 리뷰 이슈 수정. Triggers on 'typescript fix', 'ts fix', '타입스크립트 수정'."
tools: Read, Bash, Glob, Grep, Write, Edit
---

# TypeScript Fix

Fix issues found in review, prioritized by severity.

**Agents:** tak-typescript-expert, code-simplicity-reviewer

**Fix pattern references:**
- `references/common-fixes.md` — Before/After code for all common TypeScript fixes

---

## Pre-Fix

**From review output, load:**
1. Fix Checklist (Critical -> Important -> Nice-to-have)
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

### Never Do
- Skip [Critical] issues
- Change logic beyond the fix
- Introduce new patterns
- Leave verification failing

---

## Fix Priority

```
[Critical] (MUST fix - no exceptions)
    |
    |- export default -> named export (except App/config entrypoint)
    |- any -> proper types
    |- function -> arrow function
    |- unhandled null -> add guards
    '- missing barrel -> add index.ts
    |
    v
[Important] (SHOULD fix)
    |
    |- deep nesting -> early returns
    |- unnamed conditions -> named variables
    '- testability improvements
    |
    v
[Nice-to-have] (OPTIONAL)
    |
    '- simplification, naming, type inference
```

---

## Checklist

- [ ] ALL [Critical] issues fixed
- [ ] [Important] issues addressed (or justified skip)
- [ ] `{pm} run lint` passes (if available)
- [ ] `{pm} test` passes (if available)
- [ ] `{pm} run build` passes (if available)
- [ ] Fix report generated
- [ ] Re-review decision made

## Verification Commands
(see `_shared/verification-commands.md`)

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

## Re-Review Criteria

| Condition | Action |
|-----------|--------|
| Had Critical issues | -> Run `/ts-03-review` again (required) |
| Had only Important issues | -> Self-check and complete |
| Final score 8/10 or higher | -> Complete |
| Final score below 8/10 | -> Run `/ts-03-review` again |

---

## Output
(see `_shared/output-templates.md` -- Fix Output)

-> Next: `/ts-03-review` (re-review) or `/qa` or `/ship`
