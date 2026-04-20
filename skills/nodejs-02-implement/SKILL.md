---
name: nodejs-02-implement
description: "Node.js 백엔드 기능 구현. Express, Fastify. Triggers on 'nodejs implement', 'node implement', 'backend implement', 'server implement', '서버 구현', '백엔드 구현', 'api implement'."
tools: Read, Bash, Glob, Grep, Write, Edit, Agent
---

# Node.js Implement

Implement Node.js backend features (Express/Fastify). Adapts to project stack.

**Agents:** nodejs-backend-expert, api-designer, best-practices-researcher

**Refer to code patterns in `references/` directory:**
- `references/layered-architecture-pattern.md` -- Route → Service → Repository patterns

---

## Pre-Implementation

**Before coding, verify from plan output:**
1. Plan approved?
2. Files to create/modify identified?
3. Implementation Checklist ready?
4. Stack detected? (framework, ORM, validation)

---

(see `_shared/implement-common.md`)

---

## Boundaries

### Always Do
- Use plan's Implementation Checklist
- Layered architecture: Route → Service → Repository
- Input validation at HTTP boundary
- Centralized error handling (throw AppError, global handler catches)
- DI for testability
- Run verification commands before done

### [Warning] Ask First
- Adding new npm dependencies
- DB migrations
- Modifying middleware chain
- Adding queue/worker infrastructure

### Never Do
- DB queries in route handlers
- Business logic in controllers/routes
- Scattered `process.env` access (use centralized config)
- Secrets in source code
- `any` without justification
- Swallow errors silently
- Skip input validation

---

## Architecture

```
Route/Controller (HTTP layer — parse request, format response)
    |
Service (Business logic — orchestration, rules, validation)
    |
Repository (Data access — DB queries, external API calls)
    |
Database / External Service
```

---

## Folder Structure

```
src/
  modules/
    {feature}/
      {feature}.route.ts          # Route definitions
      {feature}.service.ts        # Business logic
      {feature}.repository.ts     # Data access
      {feature}.schema.ts         # Validation schemas
      {feature}.types.ts          # TypeScript types
      __tests__/
        {feature}.service.test.ts
        {feature}.route.test.ts
  common/
    middleware/                    # Auth, error handler, logging
    errors/                       # AppError, NotFoundError, etc.
    utils/
    types/
  config/
    index.ts                      # Validated env config
    database.ts
  index.ts                        # Entry point
```

---

## Verification Commands

```bash
# Detect package manager
if [ -f "bun.lockb" ]; then pm="bun"
elif [ -f "pnpm-lock.yaml" ]; then pm="pnpm"
elif [ -f "yarn.lock" ]; then pm="yarn"
else pm="npm"; fi

$pm run lint                      # Lint
$pm test                          # Tests
$pm run build                     # TypeScript compilation
$pm run test:integration          # Integration tests (if configured)
```

---

## Checklist

- [ ] Layered architecture (Route → Service → Repository)
- [ ] Input validation at HTTP boundary
- [ ] Centralized error handling
- [ ] Proper HTTP status codes (200, 201, 204, 400, 401, 403, 404, 500)
- [ ] Environment config centralized and validated
- [ ] DI for testability
- [ ] No secrets in code
- [ ] Async errors propagated (not swallowed)
- [ ] DB migrations created (if schema changed)
- [ ] Request/response types defined
- [ ] Parameterized queries (no string interpolation in SQL/NoSQL)
- [ ] Protected endpoints have auth middleware
- [ ] Mass assignment prevented (validation schemas define allowed fields)
- [ ] Named exports only
- [ ] const + arrow functions
- [ ] Verification commands pass

---

## --team Mode
(see `_shared/team-mode.md`)

### Team Composition

| Role | Responsibility | Files |
|------|---------------|-------|
| Route Agent | HTTP layer, validation schemas | `*.route.ts`, `*.schema.ts` |
| Service Agent | Business logic, orchestration | `*.service.ts` |
| Data Agent | Repository, DB queries, migrations | `*.repository.ts`, `migrations/` |
| Test Agent | Unit + integration tests | `__tests__/` |

---

## Output
(see `_shared/output-templates.md`)

-> Next: `/simplify` → `/nodejs-03-review`
