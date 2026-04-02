---
name: nodejs-03-review
description: "Node.js 백엔드 코드 리뷰. Triggers on 'nodejs review', 'node review', 'backend review', 'server review', '서버 리뷰', '백엔드 리뷰', 'api review'."
context: fork
tools: Read, Bash, Glob, Grep
---

# Node.js Review

Review Node.js backend code and generate Fix Checklist for next phase.

**Agents:** nodejs-backend-expert, code-simplicity-reviewer

**Context: fork** -- Review in a separate context from implementation (Test-Time Compute: same model but separated context = better results).

**Smart Review Routing:** Security-related -> `/cso`, API design -> `api-designer`

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

### Never Do
- Skip Critical checks
- Give vague feedback without fix examples
- Approve code with Critical issues

---

## Checks (Priority Order)

### [Critical] (Auto-Fail)
1. DB queries in route handler (bypassing service/repository)?
2. Business logic in controller/route?
3. Unvalidated external input (no schema validation)?
4. Secrets/credentials hardcoded in source?
5. Missing error handling (swallowed errors, no global handler)?
6. SQL injection / NoSQL injection risk?
7. Missing auth on protected endpoint?
8. Mass assignment risk (spreading req.body directly into DB create/update)?
9. SSRF risk (user-controlled URLs passed to fetch/http without validation)?

### [Important]
10. Missing layered architecture (Route → Service → Repository)?
11. Scattered `process.env` access (no centralized config)?
12. Direct concrete dependency (no DI, hard to test)?
13. Missing proper HTTP status codes?
14. No request logging?
15. Missing graceful shutdown?
16. N+1 query pattern?

### [Nice-to-have]
17. Could service be simplified?
18. Missing pagination on list endpoints?
19. Response shape inconsistent?

---

## Decision Tree

```
DB query in route? -> [Critical] Move to repository via service
Business logic in route? -> [Critical] Move to service
Unvalidated input? -> [Critical] Add Zod/AJV schema
Hardcoded secrets? -> [Critical] Move to env config
Missing error handling? -> [Critical] Add AppError + global handler
Injection risk? -> [Critical] Use parameterized queries
Missing auth? -> [Critical] Add auth middleware
Mass assignment? -> [Critical] Use explicit schema, don't spread req.body
SSRF risk? -> [Critical] Validate/allowlist URLs
No layered arch? -> [Important] Restructure to Route→Service→Repo
Scattered env access? -> [Important] Centralize config
No DI? -> [Important] Factory injection
Wrong status codes? -> [Important] Fix (201 create, 204 delete, etc.)
No logging? -> [Important] Add request logger
No graceful shutdown? -> [Important] Add SIGTERM/SIGINT handler
All pass -> Score 10/10
```

---

## Scoring
(see `_shared/scoring-table.md`)

## Verification Commands

```bash
# Detect package manager
if [ -f "bun.lockb" ]; then pm="bun"
elif [ -f "pnpm-lock.yaml" ]; then pm="pnpm"
elif [ -f "yarn.lock" ]; then pm="yarn"
else pm="npm"; fi

$pm run lint
$pm test
$pm run build
```

---

## --team Mode
(see `_shared/team-mode.md`)

### Team Composition

| Role | Focus | Agent |
|------|-------|-------|
| Backend Expert | Architecture, patterns, security | nodejs-backend-expert |
| Code Quality | Complexity, naming, YAGNI | code-simplicity-reviewer |

---

## Output
(see `_shared/output-templates.md` -- Review Output)

-> If issues exist: `/nodejs-04-fix`, otherwise: `/qa` or `/ship`
