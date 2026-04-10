---
name: nodejs-05-test
description: "Node.js 백엔드 테스트 작성 및 실행. Triggers on 'nodejs test', 'node test', 'backend test', 'server test', '서버 테스트', '백엔드 테스트', 'api test', 'supertest'."
model: sonnet
tools: Read, Bash, Glob, Grep, Write, Edit
---

# Node.js Test

Write and run tests for Node.js backend features.

**Agents:** test-strategy-expert, nodejs-backend-expert

## Pre-Test

1. Implementation complete? (`/nodejs-02-implement`)
2. Review passed? (`/nodejs-03-review` -> `/nodejs-04-fix`)
3. Files to test identified

---

## Test Strategy

1. **Analyze implemented files** — prioritize by criticality:
   - [Critical] Service, Repository, Route/Controller, Utility
   - [Important] Middleware, Validation schema

2. **Mocking targets**: Repository (for service unit tests), External APIs (MSW), Database (in-memory/Testcontainers), Cache (ioredis-mock)

---

## Rules

- **Do**: AAA pattern, behavior-based naming (`should [action] when [condition]`), test error/edge cases, test HTTP status codes, factory functions for test data, DB cleanup between tests
- **Don't**: test implementation details, share state between tests, use `sleep()`/`setTimeout()`, mock what you don't own, hardcode test data inline

**Maintain 100% compatibility with existing test stack and patterns.**

---

## Run and Verify

```bash
if [ -f "bun.lockb" ]; then pm="bun"; elif [ -f "pnpm-lock.yaml" ]; then pm="pnpm"; elif [ -f "yarn.lock" ]; then pm="yarn"; else pm="npm"; fi
$pm test -- --run
$pm vitest run --coverage
```

-> All tests pass -> Complete
-> Failures -> `/nodejs-04-fix`
