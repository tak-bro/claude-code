---
name: nodejs-04-fix
description: "Node.js 백엔드 리뷰 이슈 수정. Triggers on 'nodejs fix', 'node fix', 'backend fix', 'server fix', '서버 수정', '백엔드 수정', 'api fix'."
tools: Read, Bash, Glob, Grep, Write, Edit
model: sonnet

# Node.js Fix

Fix issues found in review, prioritized by severity.

**Agents:** nodejs-backend-expert

**Fix pattern references:**
- `references/common-fixes.md` -- Before/After code for all common backend fixes
- `../nodejs-02-implement/references/layered-architecture-pattern.md`

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
    |-- DB query in route -> move to repository via service
    |-- Business logic in route -> move to service
    |-- Unvalidated input -> add validation schema
    |-- Hardcoded secrets -> move to env config
    |-- Missing error handling -> add AppError + global handler
    |-- Injection risk -> parameterized queries
    |-- Missing auth -> add middleware
    |-- Mass assignment -> use explicit schema, don't spread req.body
    |-- SSRF risk -> validate/allowlist URLs
    |
    v
[Important] (SHOULD fix)
    |
    |-- No layered architecture
    |-- Scattered env access -> centralize config
    |-- No DI -> factory injection
    |-- Wrong status codes
    |-- No logging
    |-- No graceful shutdown
    |
    v
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
- [ ] Lint passes
- [ ] Tests pass
- [ ] Build passes
- [ ] Fix report generated
- [ ] Re-review decision made

## Re-Review Criteria

| Condition | Action |
|-----------|--------|
| Had Critical issues | -> Run `/nodejs-03-review` again (required) |
| Had only Important issues | -> Self-check and complete |
| Final score 8/10 or higher | -> Complete |
| Final score below 8/10 | -> Run `/nodejs-03-review` again |

---

## Output
(see `_shared/output-templates.md` -- Fix Output)

-> Next: `/nodejs-03-review` (re-review) or `/qa` or `/ship`
