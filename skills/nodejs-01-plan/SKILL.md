---
name: nodejs-01-plan
description: "Node.js 백엔드 구현 계획. Triggers on 'nodejs plan', 'node plan', 'backend plan', 'server plan', '서버 계획', '백엔드 계획', 'api plan'. Express, Fastify."
tools: Read, Bash, Glob, Grep, Agent
---

# Node.js Plan

Node.js backend implementation planning. **Base: Reference `/plan` skill for same output structure.**

## CRITICAL: STOP AFTER PLAN
Stop after planning is complete. Wait for the user's next command.
- **Plan file path:** `.claude/{YYYYMMDD}/PLAN-{HHMMSS}.md`
- Output: `Plan complete -> Next: /nodejs-02-implement`

---

## Phase 0: Explore First
Read the code first. Parallel exploration via sub-agents:
- `codebase-researcher`: Analyze existing route/service patterns, folder structure
- `best-practices-researcher`: Verify framework docs (Express/Fastify/NestJS)
- `nodejs-backend-expert`: Check architecture, DI setup, error handling patterns

### Stack Detection (CRITICAL)
```bash
# Detect framework, ORM, validation, test stack from package.json
cat package.json | grep -E "express|fastify|@nestjs|prisma|typeorm|drizzle|mongoose|mongodb|zod|ajv|class-validator|vitest|jest|supertest"
```

## Phase 0.5: Rethink
For new features: "Is this really what the user wants?" Redefine (skip for bug fixes/infrastructure)

---

## Knowledge Sources
- `.claude/llms.txt`: [Project architecture -- generated via `/generate-codebase-context`. Reference if present]
- `./CLAUDE.md`: [patterns found]
- `[reference file]`: [patterns extracted]

---

## Backend-Specific Planning

### Phase 1 Additional Analysis

1. **Module Structure**
   - Which feature module does this belong to?
   - Need new module? (`src/modules/{feature}/`)
   - Shared dependencies from `src/common/`?

2. **API Design**
   - Endpoints (method, path, request/response shape)
   - Authentication/authorization requirements
   - Rate limiting needed?
   - Pagination strategy (cursor vs offset)

3. **Data Layer Design**
   - DB schema changes (migration needed?)
   - Repository interface definition
   - Relationships (SQL) or embedded docs (NoSQL)
   - Indexes needed for query patterns?
   - Caching strategy (Redis, in-memory)?

4. **Service Layer Design**
   - Business rules and validation
   - External service integrations
   - Background jobs / queue needed?
   - Transaction boundaries

### Phase 2 Additional Research
- Analyze existing route/service patterns (codebase-researcher)
- Check middleware chain setup
- Verify error handling patterns
- Check test patterns and coverage

---

## Backend-Specific Boundaries

### Do
- Layered architecture (Route → Service → Repository)
- Input validation at HTTP boundary
- Centralized error handling
- Plan DB schema + migrations
- Design for testability (DI)

### [Warning] Ask First
- Adding new npm dependencies
- DB schema migrations on production data
- Adding background job infrastructure (Bull, etc.)
- Modifying auth/middleware chain
- Adding new external service integrations

### Don't
- DB queries in route handlers
- Business logic in controllers
- Scattered process.env access
- God files (route + logic + DB in one)
- Secrets in source code

---

## Backend-Specific Checklist Items

- [ ] Endpoints defined (method, path, request/response)
- [ ] DB schema changes planned (migration if needed)
- [ ] Input validation schemas defined
- [ ] Error cases identified
- [ ] Auth/authz requirements specified
- [ ] Service layer responsibilities clear
- [ ] Repository interfaces defined
- [ ] Test strategy (unit + integration)
- [ ] Caching strategy (if needed)
- [ ] Rate limiting (if needed)

---

## Review Focus Additional Items

- [ ] Layered architecture respected
- [ ] No DB queries in route handlers
- [ ] Input validation present
- [ ] Error handling centralized
- [ ] Config centralized (no scattered process.env)
- [ ] DI used for testability

## Plan Mode Usage
- Enter Plan Mode with Shift+Tab -> iterate 2-3 times
- Use `think hard` for API design and data modeling decisions

-> Next: `/nodejs-02-implement`
