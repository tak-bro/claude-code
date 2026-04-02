# Common Node.js Backend Fixes

## [Critical] DB Query in Route Handler

```typescript
// Before (BAD)
router.get('/users/:id', async (req, res) => {
  const user = await prisma.user.findUnique({ where: { id: req.params.id } });
  res.json(user);
});

// After (GOOD)
router.get('/users/:id', async (req, res, next) => {
  try {
    const user = await userService.getById(req.params.id);
    res.json({ data: user });
  } catch (err) {
    next(err);
  }
});
```

## [Critical] Business Logic in Route

```typescript
// Before (BAD)
router.post('/orders', async (req, res) => {
  const user = await userRepo.findById(req.body.userId);
  const discount = user.tier === 'gold' ? 0.2 : 0.1;
  const total = req.body.amount * (1 - discount);
  const order = await orderRepo.create({ ...req.body, total });
  await emailService.send(user.email, `Order total: ${total}`);
  res.status(201).json(order);
});

// After (GOOD)
router.post('/orders', validate({ body: createOrderSchema }), async (req, res, next) => {
  try {
    const order = await orderService.create(req.body);
    res.status(201).json({ data: order });
  } catch (err) {
    next(err);
  }
});
```

## [Critical] Unvalidated Input

```typescript
// Before (BAD)
router.post('/users', async (req, res) => {
  const user = await userService.create(req.body); // Raw, unvalidated!
  res.json(user);
});

// After (GOOD) - Zod
const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
});

router.post('/users', validate({ body: createUserSchema }), async (req, res, next) => {
  try {
    const user = await userService.create(req.body); // Validated
    res.status(201).json({ data: user });
  } catch (err) {
    next(err);
  }
});
```

## [Critical] Hardcoded Secrets

```typescript
// Before (BAD)
const jwt = require('jsonwebtoken');
const token = jwt.sign(payload, 'my-super-secret-key');

const dbUrl = 'mongodb://admin:password123@localhost:27017/mydb';

// After (GOOD)
import { config } from '../../config';

const token = jwt.sign(payload, config.JWT_SECRET);
// config.ts validates DATABASE_URL exists at startup
```

## [Critical] Missing Error Handling

```typescript
// Before (BAD) - try-catch in every handler
router.get('/users/:id', async (req, res) => {
  try {
    const user = await userService.getById(req.params.id);
    res.json(user);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Something went wrong' });
  }
});

// After (GOOD) - Throw custom errors + global handler catches
// service.ts
const getById = async (id: string): Promise<User> => {
  const user = await userRepository.findById(id);
  if (!user) throw new NotFoundError('User not found');
  return user;
};

// route.ts
router.get('/users/:id', async (req, res, next) => {
  try {
    const user = await userService.getById(req.params.id);
    res.json({ data: user });
  } catch (err) {
    next(err); // Global error handler handles it
  }
});

// app.ts
app.use(errorHandler); // Centralized
```

## [Critical] SQL/NoSQL Injection

```typescript
// Before (BAD) - String interpolation
const users = await db.query(`SELECT * FROM users WHERE name = '${req.query.name}'`);

// Before (BAD) - MongoDB injection
const user = await User.findOne({ email: req.body.email }); // req.body.email could be { $gt: "" }

// After (GOOD) - Parameterized query
const users = await db.query('SELECT * FROM users WHERE name = $1', [req.query.name]);

// After (GOOD) - Validated input
const schema = z.object({ email: z.string().email() });
const { email } = schema.parse(req.body);
const user = await User.findOne({ email });
```

## [Critical] Missing Auth

```typescript
// Before (BAD)
router.delete('/users/:id', async (req, res, next) => {
  await userService.delete(req.params.id); // Anyone can delete!
  res.status(204).send();
});

// After (GOOD)
router.delete('/users/:id', authenticate, authorize('admin'), async (req, res, next) => {
  try {
    await userService.delete(req.params.id);
    res.status(204).send();
  } catch (err) {
    next(err);
  }
});
```

## [Important] Scattered process.env

```typescript
// Before (BAD) - process.env everywhere
// file1.ts
const port = process.env.PORT || 3000;
// file2.ts
const dbUrl = process.env.DATABASE_URL!;
// file3.ts
if (process.env.NODE_ENV === 'production') { ... }

// After (GOOD) - Centralized config
// config/index.ts
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
});

export const config = envSchema.parse(process.env);

// All other files
import { config } from '../config';
const port = config.PORT;
```

## [Important] No DI (Hard to Test)

```typescript
// Before (BAD)
import { prisma } from '../db';

export const getUser = async (id: string) => {
  return prisma.user.findUnique({ where: { id } }); // Can't mock in tests!
};

// After (GOOD)
export const createUserRepository = (prisma: PrismaClient) => ({
  findById: (id: string) => prisma.user.findUnique({ where: { id } }),
});

// In tests: createUserRepository(mockPrisma)
```

## [Important] Wrong HTTP Status Codes

```typescript
// Before (BAD)
router.post('/users', async (req, res) => {
  const user = await userService.create(req.body);
  res.json(user);        // 200 for creation? Should be 201
});

router.delete('/users/:id', async (req, res) => {
  await userService.delete(req.params.id);
  res.json({ ok: true }); // Should be 204 No Content
});

// After (GOOD)
router.post('/users', async (req, res, next) => {
  try {
    const user = await userService.create(req.body);
    res.status(201).json({ data: user });
  } catch (err) {
    next(err);
  }
});

router.delete('/users/:id', async (req, res, next) => {
  try {
    await userService.delete(req.params.id);
    res.status(204).send();
  } catch (err) {
    next(err);
  }
});
```

## [Important] N+1 Query

```typescript
// Before (BAD)
const users = await userRepo.findMany();
const usersWithPosts = await Promise.all(
  users.map(async (user) => ({
    ...user,
    posts: await postRepo.findByUserId(user.id), // N queries!
  })),
);

// After (GOOD) - Prisma
const users = await prisma.user.findMany({ include: { posts: true } });

// After (GOOD) - MongoDB
const pipeline = [
  { $lookup: { from: 'posts', localField: '_id', foreignField: 'userId', as: 'posts' } },
];
const users = await UserModel.aggregate(pipeline);

// After (GOOD) - SQL
const users = await db.query(`
  SELECT u.*, json_agg(p.*) as posts
  FROM users u LEFT JOIN posts p ON p.user_id = u.id
  GROUP BY u.id
`);
```
