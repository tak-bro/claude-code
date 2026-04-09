---
name: nodejs-05-test
model: sonnet
description: "Node.js 백엔드 테스트 작성 및 실행. Triggers on 'nodejs test', 'node test', 'backend test', 'server test', '서버 테스트', '백엔드 테스트', 'api test', 'supertest'."
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

## Node.js Test Stack

```
Vitest / Jest                    # Test framework
Supertest                        # HTTP integration testing
MSW (Mock Service Worker)        # External API mocking
Testcontainers (optional)        # DB in Docker for integration
In-memory DB alternatives        # SQLite (Prisma), mongodb-memory-server
```

**Maintain 100% compatibility with existing test stack and patterns.**

---

## Test Strategy

1. **Analyze implemented files**
   - Service -> [Critical] Required test
   - Repository -> [Critical] Required test (integration with DB)
   - Route/Controller -> [Critical] Required test (HTTP integration)
   - Middleware -> [Important] Required test
   - Validation schema -> [Important] Required test
   - Utility/helper -> [Critical] Required test (pure functions)

2. **Determine mocking targets**
   - Repository -> Mock for service unit tests
   - External APIs -> MSW or manual mock
   - Database -> In-memory DB or Testcontainers
   - Email/SMS service -> Mock
   - Cache (Redis) -> Mock or ioredis-mock

---

## Test Patterns

### Service Unit Test (Required)

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { createUserService } from './user.service';
import { NotFoundError, ConflictError } from '../../common/errors';

describe('UserService', () => {
  const mockUserRepository = {
    findById: vi.fn(),
    findByEmail: vi.fn(),
    create: vi.fn(),
    update: vi.fn(),
    delete: vi.fn(),
  };

  const userService = createUserService({
    userRepository: mockUserRepository,
  });

  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe('getById', () => {
    it('should return user when found', async () => {
      // Arrange
      const mockUser = { id: '1', email: 'test@test.com', name: 'Test' };
      mockUserRepository.findById.mockResolvedValue(mockUser);

      // Act
      const result = await userService.getById('1');

      // Assert
      expect(result).toEqual(mockUser);
      expect(mockUserRepository.findById).toHaveBeenCalledWith('1');
    });

    it('should throw NotFoundError when user not found', async () => {
      // Arrange
      mockUserRepository.findById.mockResolvedValue(null);

      // Act & Assert
      await expect(userService.getById('999')).rejects.toThrow(NotFoundError);
    });
  });

  describe('create', () => {
    it('should create user when email is unique', async () => {
      // Arrange
      const body = { email: 'new@test.com', name: 'New' };
      mockUserRepository.findByEmail.mockResolvedValue(null);
      mockUserRepository.create.mockResolvedValue({ id: '1', ...body });

      // Act
      const result = await userService.create(body);

      // Assert
      expect(result).toMatchObject(body);
    });

    it('should throw ConflictError when email exists', async () => {
      // Arrange
      mockUserRepository.findByEmail.mockResolvedValue({ id: '1' });

      // Act & Assert
      await expect(
        userService.create({ email: 'exists@test.com', name: 'Dup' }),
      ).rejects.toThrow(ConflictError);
    });
  });
});
```

### Route Integration Test (Required)

```typescript
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import request from 'supertest';
import { createApp } from '../../app';

// Test helpers (define once, reuse across test files)
const createTestContext = async () => {
  const app = await createApp({ /* test config */ });
  const testToken = 'test-jwt-token';       // from test auth setup
  const adminToken = 'test-admin-token';
  return { app, testToken, adminToken };
};

describe('GET /api/users', () => {
  let app: Express;
  let testToken: string;

  beforeAll(async () => {
    const ctx = await createTestContext();
    app = ctx.app;
    testToken = ctx.testToken;
  });

  it('should return 200 with users list', async () => {
    const response = await request(app)
      .get('/api/users')
      .set('Authorization', `Bearer ${testToken}`)
      .expect(200);

    expect(response.body.data).toBeInstanceOf(Array);
  });

  it('should return 401 without auth token', async () => {
    await request(app)
      .get('/api/users')
      .expect(401);
  });
});

describe('POST /api/users', () => {
  let app: Express;
  let adminToken: string;

  beforeAll(async () => {
    const ctx = await createTestContext();
    app = ctx.app;
    adminToken = ctx.adminToken;
  });

  it('should return 201 with created user', async () => {
    const body = { email: 'new@test.com', name: 'New User' };

    const response = await request(app)
      .post('/api/users')
      .set('Authorization', `Bearer ${adminToken}`)
      .send(body)
      .expect(201);

    expect(response.body.data).toMatchObject(body);
  });

  it('should return 400 with invalid email', async () => {
    const response = await request(app)
      .post('/api/users')
      .set('Authorization', `Bearer ${adminToken}`)
      .send({ email: 'invalid', name: 'Test' })
      .expect(400);

    expect(response.body.error.code).toBe('VALIDATION_ERROR');
  });

  it('should return 409 when email already exists', async () => {
    const body = { email: 'existing@test.com', name: 'Dup' };
    // First create
    await request(app).post('/api/users').set('Authorization', `Bearer ${adminToken}`).send(body);

    // Duplicate
    await request(app)
      .post('/api/users')
      .set('Authorization', `Bearer ${adminToken}`)
      .send(body)
      .expect(409);
  });
});

describe('DELETE /api/users/:id', () => {
  let app: Express;
  let adminToken: string;

  beforeAll(async () => {
    const ctx = await createTestContext();
    app = ctx.app;
    adminToken = ctx.adminToken;
  });

  it('should return 204 on successful delete', async () => {
    // Arrange: create a user first
    const created = await request(app)
      .post('/api/users')
      .set('Authorization', `Bearer ${adminToken}`)
      .send({ email: 'delete-me@test.com', name: 'Delete Me' });

    // Act & Assert
    await request(app)
      .delete(`/api/users/${created.body.data.id}`)
      .set('Authorization', `Bearer ${adminToken}`)
      .expect(204);
  });

  it('should return 404 when user not found', async () => {
    await request(app)
      .delete('/api/users/nonexistent')
      .set('Authorization', `Bearer ${adminToken}`)
      .expect(404);
  });
});
```

### Validation Schema Test (Important)

```typescript
import { describe, it, expect } from 'vitest';
import { createUserSchema } from './user.schema';

describe('createUserSchema', () => {
  it('should accept valid input', () => {
    const result = createUserSchema.safeParse({
      email: 'test@test.com',
      name: 'Test User',
    });
    expect(result.success).toBe(true);
  });

  it('should reject invalid email', () => {
    const result = createUserSchema.safeParse({
      email: 'not-an-email',
      name: 'Test',
    });
    expect(result.success).toBe(false);
  });

  it('should reject empty name', () => {
    const result = createUserSchema.safeParse({
      email: 'test@test.com',
      name: '',
    });
    expect(result.success).toBe(false);
  });
});
```

---

## Test Writing Rules

### Do
- Behavior-based test naming (`should [action] when [condition]`)
- AAA pattern (Arrange-Act-Assert) required
- Test error/edge cases, not just happy path
- Test HTTP status codes explicitly
- Test validation rejections
- Use factory functions for test data
- Clean up test DB state between tests (see DB Cleanup below)

### Don't
- Test implementation details (internal method calls)
- Share state between tests
- Use `sleep()` / `setTimeout()` for timing
- Mock what you don't own (mock your adapters instead)
- Skip integration tests (they catch what unit tests miss)
- Hardcode test data inline (use factories)

---

## DB Cleanup Patterns

### Transaction rollback (Prisma)
```typescript
import { PrismaClient } from '@prisma/client';

let prisma: PrismaClient;

beforeAll(async () => {
  prisma = new PrismaClient({ datasourceUrl: process.env.TEST_DATABASE_URL });
});

afterAll(async () => {
  await prisma.$disconnect();
});

beforeEach(async () => {
  // Truncate all tables between tests
  const tablenames = await prisma.$queryRaw<{ tablename: string }[]>`
    SELECT tablename FROM pg_tables WHERE schemaname = 'public'
  `;
  for (const { tablename } of tablenames) {
    if (tablename !== '_prisma_migrations') {
      await prisma.$executeRawUnsafe(`TRUNCATE TABLE "${tablename}" CASCADE`);
    }
  }
});
```

### In-memory MongoDB (mongodb-memory-server)
```typescript
import { MongoMemoryServer } from 'mongodb-memory-server';
import { MongoClient } from 'mongodb';

let mongod: MongoMemoryServer;
let client: MongoClient;

beforeAll(async () => {
  mongod = await MongoMemoryServer.create();
  client = await MongoClient.connect(mongod.getUri());
});

afterAll(async () => {
  await client.close();
  await mongod.stop();
});

beforeEach(async () => {
  const collections = await client.db().collections();
  await Promise.all(collections.map((c) => c.deleteMany({})));
});
```

---

## Run and Verify

```bash
# Detect package manager
if [ -f "bun.lockb" ]; then pm="bun"
elif [ -f "pnpm-lock.yaml" ]; then pm="pnpm"
elif [ -f "yarn.lock" ]; then pm="yarn"
else pm="npm"; fi

# Unit tests
$pm test -- --run
$pm vitest run --reporter=verbose src/modules/{feature}/

# Integration tests
$pm run test:integration

# Coverage
$pm vitest run --coverage

# Specific file
$pm vitest run src/modules/user/__tests__/user.service.test.ts
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

-> All tests pass -> Complete
-> Failures -> Run `/nodejs-04-fix` again
```
