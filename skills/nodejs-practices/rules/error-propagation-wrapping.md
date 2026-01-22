---
title: Error Propagation and Wrapping
impact: MEDIUM
impactDescription: preserves error context while providing clean user-facing messages
tags: error-handling, error-cause, error-wrapping, debugging
---

## Error Propagation and Wrapping

Preserve error context when rethrowing. Use the `cause` option (ES2022+) to chain errors for debugging while keeping user-facing messages clean.

**Incorrect (losing original error):**

```typescript
// Original error and stack trace lost
async function getUser(id: string) {
  try {
    return await db.users.findById(id);
  } catch (err) {
    throw new Error("Failed to get user"); // What actually went wrong?
  }
}

// Exposing internal details to users
async function createOrder(data: OrderData) {
  try {
    return await db.orders.insert(data);
  } catch (err) {
    throw err; // Might expose: "UNIQUE constraint failed: orders.reference_id"
  }
}

// Concatenating messages loses structure
catch (err) {
  throw new Error(`Database error: ${err.message}`);
}
```

**Correct (preserving cause chain):**

```typescript
// Use cause option to preserve original error
async function getUser(id: string) {
  try {
    return await db.users.findById(id);
  } catch (err) {
    throw new AppError(
      "Failed to get user",
      500,
      "DATABASE_ERROR",
      { userId: id },
      { cause: err }  // ES2022 error cause
    );
  }
}

// Wrap with appropriate abstraction level
async function createOrder(data: OrderData) {
  try {
    return await db.orders.insert(data);
  } catch (err) {
    if (isUniqueConstraintError(err)) {
      throw new ConflictError("Order reference already exists", {
        cause: err
      });
    }
    throw new AppError("Failed to create order", 500, "DATABASE_ERROR", {}, {
      cause: err
    });
  }
}
```

**AppError with cause support:**

```typescript
class AppError extends Error {
  readonly isOperational = true;

  constructor(
    message: string,
    readonly statusCode: number,
    readonly code: string,
    readonly metadata?: Record<string, unknown>,
    options?: ErrorOptions  // { cause?: Error }
  ) {
    super(message, options);
    this.name = this.constructor.name;
  }
}

// Usage
throw new AppError("User fetch failed", 500, "DB_ERROR", { id }, { cause: err });
```

**Logging the cause chain:**

```typescript
function logErrorChain(err: Error, depth = 0): void {
  const indent = "  ".repeat(depth);
  console.error(`${indent}${err.name}: ${err.message}`);

  if (err.stack) {
    const stackLines = err.stack.split("\n").slice(1, 4);
    stackLines.forEach(line => console.error(`${indent}${line}`));
  }

  if (err.cause instanceof Error) {
    console.error(`${indent}Caused by:`);
    logErrorChain(err.cause, depth + 1);
  }
}

// Output:
// AppError: Failed to get user
//     at getUser (/app/users.ts:15:11)
//   Caused by:
//   MongoError: Connection refused
//       at Connection.connect (/node_modules/mongodb/...)
```

**Layer-appropriate wrapping:**

```typescript
// Repository layer - wrap DB errors
class UserRepository {
  async findById(id: string): Promise<User> {
    try {
      return await this.db.users.findById(id);
    } catch (err) {
      throw new DatabaseError("User query failed", { cause: err });
    }
  }
}

// Service layer - wrap with business context
class UserService {
  async getUser(id: string): Promise<User> {
    try {
      const user = await this.repo.findById(id);
      if (!user) throw new NotFoundError("User", id);
      return user;
    } catch (err) {
      if (err instanceof NotFoundError) throw err;
      throw new ServiceError("Failed to retrieve user", { cause: err });
    }
  }
}

// Controller - let errors propagate to error handler
class UserController {
  async get(req: Request, res: Response) {
    const user = await this.service.getUser(req.params.id);
    res.json(user);
    // Errors propagate to global error handler
  }
}
```

Each layer should add context appropriate to its abstraction level while preserving the underlying cause.
