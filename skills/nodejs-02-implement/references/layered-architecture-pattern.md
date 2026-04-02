# Layered Architecture Pattern Reference

## asyncHandler (Express utility)

```typescript
// Eliminates try-catch boilerplate in every Express handler
import { RequestHandler } from 'express';

export const asyncHandler = (fn: (...args: Parameters<RequestHandler>) => Promise<void>): RequestHandler =>
  (req, res, next) => fn(req, res, next).catch(next);
```

## Route Layer (Express)

```typescript
import { Router } from 'express';
import { UserService } from './user.service';
import { createUserSchema, updateUserSchema, userParamsSchema } from './user.schema';
import { validate } from '../../common/middleware/validate';
import { asyncHandler } from '../../common/middleware/asyncHandler';

export const createUserRouter = (userService: UserService): Router => {
  const router = Router();

  router.get('/', asyncHandler(async (req, res) => {
    const users = await userService.findMany(req.query);
    res.json({ data: users });
  }));

  router.get('/:id', validate({ params: userParamsSchema }), asyncHandler(async (req, res) => {
    const user = await userService.getById(req.params.id);
    res.json({ data: user });
  }));

  router.post('/', validate({ body: createUserSchema }), asyncHandler(async (req, res) => {
    const user = await userService.create(req.body);
    res.status(201).json({ data: user });
  }));

  router.patch('/:id', validate({ params: userParamsSchema, body: updateUserSchema }), asyncHandler(async (req, res) => {
    const user = await userService.update(req.params.id, req.body);
    res.json({ data: user });
  }));

  router.delete('/:id', validate({ params: userParamsSchema }), asyncHandler(async (req, res) => {
    await userService.delete(req.params.id);
    res.status(204).send();
  }));

  return router;
};
```

## Route Layer (Fastify)

```typescript
import { FastifyPluginAsync } from 'fastify';
import { UserService } from './user.service';
import { CreateUserBody, UpdateUserBody } from './user.types';

export const userRoutes = (userService: UserService): FastifyPluginAsync =>
  async (fastify) => {
    fastify.get('/', async (request) => {
      const users = await userService.findMany(request.query);
      return { data: users };
    });

    fastify.get<{ Params: { id: string } }>('/:id', {
      schema: {
        params: { type: 'object', properties: { id: { type: 'string' } }, required: ['id'] },
      },
    }, async (request) => {
      const user = await userService.getById(request.params.id);
      return { data: user };
    });

    fastify.post<{ Body: CreateUserBody }>('/', {
      schema: {
        body: {
          type: 'object',
          required: ['email', 'name'],
          properties: {
            email: { type: 'string', format: 'email' },
            name: { type: 'string', minLength: 1 },
          },
        },
      },
    }, async (request, reply) => {
      const user = await userService.create(request.body);
      reply.status(201);
      return { data: user };
    });
  };
```

## Service Layer

```typescript
import { UserRepository } from './user.repository';
import { NotFoundError, ConflictError } from '../../common/errors';
import { CreateUserBody, UpdateUserBody, User } from './user.types';

export type UserService = ReturnType<typeof createUserService>;

export const createUserService = (deps: {
  userRepository: UserRepository;
  emailService?: EmailService;
}) => {
  const getById = async (id: string): Promise<User> => {
    const user = await deps.userRepository.findById(id);
    if (!user) throw new NotFoundError('User not found');
    return user;
  };

  const create = async (body: CreateUserBody): Promise<User> => {
    const existing = await deps.userRepository.findByEmail(body.email);
    if (existing) throw new ConflictError('Email already in use');

    const user = await deps.userRepository.create(body);
    await deps.emailService?.sendWelcome(user.email);
    return user;
  };

  const update = async (id: string, body: UpdateUserBody): Promise<User> => {
    await getById(id); // throws NotFoundError if missing
    return deps.userRepository.update(id, body);
  };

  const remove = async (id: string): Promise<void> => {
    await getById(id);
    await deps.userRepository.delete(id);
  };

  const findMany = async (filter?: UserFilter): Promise<User[]> => {
    return deps.userRepository.findMany(filter);
  };

  return { getById, create, update, delete: remove, findMany };
};
```

## Repository Layer (Prisma)

```typescript
import { PrismaClient } from '@prisma/client';
import { CreateUserBody, UpdateUserBody, User, UserFilter } from './user.types';

export type UserRepository = ReturnType<typeof createUserRepository>;

export const createUserRepository = (prisma: PrismaClient) => ({
  findById: (id: string): Promise<User | null> =>
    prisma.user.findUnique({ where: { id } }),

  findByEmail: (email: string): Promise<User | null> =>
    prisma.user.findUnique({ where: { email } }),

  create: (data: CreateUserBody): Promise<User> =>
    prisma.user.create({ data }),

  update: (id: string, data: UpdateUserBody): Promise<User> =>
    prisma.user.update({ where: { id }, data }),

  delete: async (id: string): Promise<void> => {
    await prisma.user.delete({ where: { id } });
  },

  findMany: (filter?: UserFilter): Promise<User[]> =>
    prisma.user.findMany({
      where: filter,
      orderBy: { createdAt: 'desc' },
    }),
});
```

## Repository Layer (Mongoose)

```typescript
import { Model, FilterQuery } from 'mongoose';
import { UserDocument } from './user.model';
import { CreateUserBody, UpdateUserBody, User } from './user.types';

export type UserRepository = ReturnType<typeof createUserRepository>;

export const createUserRepository = (UserModel: Model<UserDocument>) => ({
  findById: (id: string): Promise<User | null> =>
    UserModel.findById(id).lean().exec(),

  findByEmail: (email: string): Promise<User | null> =>
    UserModel.findOne({ email }).lean().exec(),

  create: (data: CreateUserBody): Promise<User> =>
    UserModel.create(data).then((doc) => doc.toObject()),

  update: (id: string, data: UpdateUserBody): Promise<User | null> =>
    UserModel.findByIdAndUpdate(id, { $set: data }, { new: true }).lean().exec(),

  delete: async (id: string): Promise<void> => {
    await UserModel.findByIdAndDelete(id).exec();
  },

  findMany: (filter?: FilterQuery<UserDocument>): Promise<User[]> =>
    UserModel.find(filter ?? {}).sort({ createdAt: -1 }).lean().exec(),
});
```

## Repository Layer (MongoDB Native)

```typescript
import { Db, ObjectId } from 'mongodb';
import { CreateUserBody, UpdateUserBody, User } from './user.types';

export type UserRepository = ReturnType<typeof createUserRepository>;

export const createUserRepository = (db: Db) => {
  const collection = db.collection<User>('users');

  return {
    findById: (id: string) =>
      collection.findOne({ _id: new ObjectId(id) }),

    findByEmail: (email: string) =>
      collection.findOne({ email }),

    create: async (data: CreateUserBody) => {
      const doc = { ...data, createdAt: new Date(), updatedAt: new Date() };
      const result = await collection.insertOne(doc);
      return { _id: result.insertedId, ...doc };
    },

    update: async (id: string, data: UpdateUserBody) =>
      collection.findOneAndUpdate(
        { _id: new ObjectId(id) },
        { $set: { ...data, updatedAt: new Date() } },
        { returnDocument: 'after' },
      ),

    delete: async (id: string) => {
      await collection.deleteOne({ _id: new ObjectId(id) });
    },

    findMany: (filter = {}) =>
      collection.find(filter).sort({ createdAt: -1 }).toArray(),
  };
};
```

## Validation Schema (Zod)

```typescript
import { z } from 'zod';

export const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  role: z.enum(['user', 'admin']).default('user'),
});

export const updateUserSchema = z.object({
  name: z.string().min(1).max(100).optional(),
  role: z.enum(['user', 'admin']).optional(),
});

export const userParamsSchema = z.object({
  id: z.string().min(1),
});

export type CreateUserBody = z.infer<typeof createUserSchema>;
export type UpdateUserBody = z.infer<typeof updateUserSchema>;
```

## Validation Middleware (Zod + Express)

```typescript
import { RequestHandler } from 'express';
import { AnyZodObject, ZodError } from 'zod';
import { ValidationError } from '../errors';

interface ValidateSchemas {
  body?: AnyZodObject;
  params?: AnyZodObject;
  query?: AnyZodObject;
}

export const validate = (schemas: ValidateSchemas): RequestHandler =>
  (req, _res, next) => {
    try {
      if (schemas.body) req.body = schemas.body.parse(req.body);
      if (schemas.params) req.params = schemas.params.parse(req.params) as any;
      if (schemas.query) req.query = schemas.query.parse(req.query) as any;
      next();
    } catch (err) {
      if (err instanceof ZodError) {
        next(new ValidationError(err.errors.map((e) => e.message).join(', ')));
        return;
      }
      next(err);
    }
  };
```

## Error Classes

```typescript
export class AppError extends Error {
  constructor(
    public readonly statusCode: number,
    message: string,
    public readonly code?: string,
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

export class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(404, message, 'NOT_FOUND');
  }
}

export class ValidationError extends AppError {
  constructor(message: string) {
    super(400, message, 'VALIDATION_ERROR');
  }
}

export class ConflictError extends AppError {
  constructor(message: string) {
    super(409, message, 'CONFLICT');
  }
}

export class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(401, message, 'UNAUTHORIZED');
  }
}

export class ForbiddenError extends AppError {
  constructor(message = 'Forbidden') {
    super(403, message, 'FORBIDDEN');
  }
}
```

## Global Error Handler (Express)

```typescript
import { ErrorRequestHandler } from 'express';
import { AppError } from '../errors';
import { logger } from '../utils/logger';

export const errorHandler: ErrorRequestHandler = (err, _req, res, _next) => {
  if (err instanceof AppError) {
    res.status(err.statusCode).json({
      error: { message: err.message, code: err.code },
    });
    return;
  }

  logger.error('Unhandled error', { error: err });
  res.status(500).json({
    error: { message: 'Internal server error', code: 'INTERNAL_ERROR' },
  });
};
```

## DI Composition Root

```typescript
// src/container.ts
import { PrismaClient } from '@prisma/client';
import { createUserRepository } from './modules/user/user.repository';
import { createUserService } from './modules/user/user.service';
import { createUserRouter } from './modules/user/user.route';

export const createContainer = (prisma: PrismaClient) => {
  // Repositories
  const userRepository = createUserRepository(prisma);

  // Services
  const userService = createUserService({ userRepository });

  // Routes
  const userRouter = createUserRouter(userService);

  return { userRouter };
};
```

## Graceful Shutdown

```typescript
import { Server } from 'http';
import { PrismaClient } from '@prisma/client';
import { logger } from './common/utils/logger';

export const setupGracefulShutdown = (server: Server, prisma: PrismaClient) => {
  const shutdown = async (signal: string) => {
    logger.info(`${signal} received, shutting down gracefully`);

    server.close(async () => {
      await prisma.$disconnect();
      logger.info('Server closed');
      process.exit(0);
    });

    setTimeout(() => {
      logger.error('Forced shutdown after timeout');
      process.exit(1);
    }, 10_000);
  };

  process.on('SIGTERM', () => shutdown('SIGTERM'));
  process.on('SIGINT', () => shutdown('SIGINT'));
};
```

## Pagination (Cursor-based)

```typescript
interface PaginationParams {
  cursor?: string;
  limit?: number;
}

interface PaginatedResult<T> {
  data: T[];
  nextCursor: string | null;
  hasMore: boolean;
}

export const paginate = async <T extends { id: string }>(
  findMany: (params: { cursor?: string; take: number }) => Promise<T[]>,
  params: PaginationParams,
): Promise<PaginatedResult<T>> => {
  const limit = Math.min(params.limit ?? 20, 100);
  const items = await findMany({ cursor: params.cursor, take: limit + 1 });

  const hasMore = items.length > limit;
  const data = hasMore ? items.slice(0, limit) : items;
  const nextCursor = hasMore ? data[data.length - 1].id : null;

  return { data, nextCursor, hasMore };
};
```
