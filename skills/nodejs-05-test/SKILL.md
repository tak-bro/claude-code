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
2. Review passed? (`/nodejs-03-review` → `/nodejs-04-fix`)
3. Files to test identified

---

## Node.js Test Stack

```
Vitest (or Jest)                  # Unit/Integration
Supertest                         # HTTP endpoint testing
MSW (Mock Service Worker)         # External API mocking
Testcontainers                    # DB integration tests
ioredis-mock                      # Redis mocking
```

**Maintain 100% compatibility with existing test stack and patterns.**

---

## Test Strategy

1. **Analyze implemented files** — prioritize by criticality:
   - [Critical] Service → business logic, error handling, edge cases
   - [Critical] Repository → query correctness, data mapping
   - [Critical] Route/Controller → status codes, validation, auth
   - [Important] Middleware → request transformation, error handling
   - [Important] Validation schema → valid/invalid input cases
   - [Nice] Utility functions (optional)

2. **Mocking targets**:
   - Repository → mock for service unit tests
   - External APIs → MSW handlers
   - Database → in-memory SQLite or Testcontainers
   - Cache → ioredis-mock
   - File system → memfs or tmp directory

---

## Test Patterns

### Service Test (Required)

```typescript
describe('FeatureService', () => {
  let service: FeatureService;
  let repository: MockFeatureRepository;

  beforeEach(() => {
    repository = createMockRepository();
    service = new FeatureService(repository);
  });

  it('should return data when found', async () => {
    // Arrange
    repository.findById.mockResolvedValue(testData);

    // Act
    const result = await service.getById('123');

    // Assert
    expect(result).toEqual(expectedData);
    expect(repository.findById).toHaveBeenCalledWith('123');
  });

  it('should throw NotFoundError when not found', async () => {
    // Arrange
    repository.findById.mockResolvedValue(null);

    // Act & Assert
    await expect(service.getById('999')).rejects.toThrow(NotFoundError);
  });

  it('should handle empty list', async () => {
    repository.findAll.mockResolvedValue([]);

    const result = await service.getAll();

    expect(result).toEqual([]);
  });
});
```

### Route/Controller Test (Required)

```typescript
describe('GET /api/features', () => {
  it('should return 200 with data', async () => {
    const response = await request(app)
      .get('/api/features')
      .set('Authorization', `Bearer ${testToken}`)
      .expect(200);

    expect(response.body.data).toHaveLength(3);
  });

  it('should return 401 without auth', async () => {
    await request(app)
      .get('/api/features')
      .expect(401);
  });

  it('should return 400 with invalid query', async () => {
    await request(app)
      .get('/api/features?limit=-1')
      .set('Authorization', `Bearer ${testToken}`)
      .expect(400);
  });
});
```

---

## Test Writing Rules

### Do
- AAA pattern (Arrange-Act-Assert) required
- Behavior-based naming (`should [action] when [condition]`)
- Test HTTP status codes explicitly (200, 201, 400, 401, 404, 500)
- Factory functions for test data (`createTestUser()`, `createTestFeature()`)
- DB cleanup between tests (`beforeEach` / `afterEach`)
- Test error/edge cases (null, empty, invalid input, duplicate)
- Test validation schemas with valid AND invalid inputs

### 🚫 Don't
- Test implementation details (internal method calls)
- Share state between tests
- Use `sleep()` / `setTimeout()` → use async/await
- Mock what you don't own (mock your abstraction instead)
- Hardcode test data inline (use factories)
- Import secrets/API keys in test files

---

## Run and Verify

```bash
if [ -f "bun.lockb" ]; then pm="bun"
elif [ -f "pnpm-lock.yaml" ]; then pm="pnpm"
elif [ -f "yarn.lock" ]; then pm="yarn"
else pm="npm"; fi

$pm test -- --run
$pm vitest run --coverage
```

---

## Output

```markdown
### Test Results: [Feature Name]

**Coverage**
| File | Statements | Branches | Functions | Lines |
|------|-----------|----------|-----------|-------|

**Test Count**
- Pass: [N]
- Fail: [N]

→ All tests pass → Complete
→ Failures → `/nodejs-04-fix`
```
