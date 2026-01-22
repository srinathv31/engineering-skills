---
title: Custom Error Classes
impact: HIGH
impactDescription: enables type-safe error handling with structured metadata
tags: error-handling, custom-errors, typescript, api
---

## Custom Error Classes

Create typed error classes with status codes and metadata for consistent, type-safe error handling.

**Incorrect (generic errors without structure):**

```typescript
// No way to distinguish error types programmatically
throw new Error("User not found");
throw new Error("Invalid email format");
throw new Error("Database connection failed");

// Using message string for control flow - brittle
try {
  await getUser(id);
} catch (err) {
  if (err.message.includes("not found")) {
    res.status(404).json({ error: err.message });
  } else if (err.message.includes("Invalid")) {
    res.status(400).json({ error: err.message });
  } else {
    res.status(500).json({ error: "Internal error" });
  }
}
```

**Correct (typed error classes):**

```typescript
// Base error class
class AppError extends Error {
  readonly isOperational = true;

  constructor(
    message: string,
    readonly statusCode: number,
    readonly code: string,
    readonly metadata?: Record<string, unknown>
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }

  toJSON() {
    return {
      code: this.code,
      message: this.message,
      ...(this.metadata && { details: this.metadata }),
    };
  }
}

// Specific error types
class NotFoundError extends AppError {
  constructor(resource: string, id?: string) {
    super(
      id ? `${resource} not found: ${id}` : `${resource} not found`,
      404,
      "NOT_FOUND",
      { resource, id }
    );
  }
}

class ValidationError extends AppError {
  constructor(errors: Record<string, string[]>) {
    super("Validation failed", 400, "VALIDATION_ERROR", { errors });
  }
}

class ConflictError extends AppError {
  constructor(message: string, field?: string) {
    super(message, 409, "CONFLICT", field ? { field } : undefined);
  }
}

class UnauthorizedError extends AppError {
  constructor(message = "Authentication required") {
    super(message, 401, "UNAUTHORIZED");
  }
}

class ForbiddenError extends AppError {
  constructor(message = "Permission denied") {
    super(message, 403, "FORBIDDEN");
  }
}
```

**Type-safe error handling:**

```typescript
// Usage
throw new NotFoundError("User", userId);
throw new ValidationError({ email: ["Invalid format"] });
throw new ConflictError("Email already registered", "email");

// Handler with type checking
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json(err.toJSON());
  }

  // Specific handling
  if (err instanceof NotFoundError) {
    logger.info("Resource not found", err.metadata);
  } else if (err instanceof ValidationError) {
    logger.debug("Validation failed", err.metadata);
  }

  // Unknown error
  logger.error("Unexpected error", err);
  res.status(500).json({ code: "INTERNAL_ERROR", message: "Internal error" });
});
```

**Error class hierarchy:**

```typescript
// Domain-specific errors
class UserError extends AppError {}
class OrderError extends AppError {}

class UserNotFoundError extends UserError {
  constructor(userId: string) {
    super(`User not found: ${userId}`, 404, "USER_NOT_FOUND", { userId });
  }
}

class OrderNotFoundError extends OrderError {
  constructor(orderId: string) {
    super(`Order not found: ${orderId}`, 404, "ORDER_NOT_FOUND", { orderId });
  }
}
```

Typed errors enable `instanceof` checks, carry structured metadata, and provide consistent API responses.
