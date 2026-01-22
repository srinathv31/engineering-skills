---
title: Operational vs Programmer Errors
impact: CRITICAL
impactDescription: ensures appropriate handling for different error categories
tags: error-handling, operational-errors, programmer-errors, node
---

## Operational vs Programmer Errors

Distinguish between operational errors (expected, recoverable) and programmer errors (bugs). Handle them with different strategies.

**Incorrect (treating all errors the same):**

```typescript
// All errors get same 500 response
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: "Something went wrong" });
});

// Catching programmer errors hides bugs
try {
  const name = user.profile.name.toUpperCase();
} catch (err) {
  return "Unknown"; // Hides the real bug: user or profile might be undefined
}

// Silently recovering from what should crash
try {
  const data = JSON.parse(internalConfig);
} catch {
  return {}; // Config parsing should never fail in production
}
```

**Correct (differentiated handling):**

```typescript
// Mark operational errors
class AppError extends Error {
  readonly isOperational = true;

  constructor(
    message: string,
    readonly statusCode: number,
    readonly code: string
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 404, "NOT_FOUND");
  }
}

class ValidationError extends AppError {
  constructor(message: string) {
    super(message, 400, "VALIDATION_ERROR");
  }
}

// Error handler differentiates
app.use((err, req, res, next) => {
  // Operational error: handle gracefully
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      code: err.code,
      message: err.message,
    });
  }

  // Programmer error: log and crash
  console.error("FATAL: Programmer error", err);
  process.exit(1);  // Let process manager restart
});
```

**Operational Errors (expected, recoverable):**

```typescript
// These are normal conditions that code should handle
- File not found
- Network timeout
- Invalid user input
- Database constraint violation
- Resource temporarily unavailable
- Authentication failure

// Handle gracefully
try {
  const user = await db.users.findById(id);
  if (!user) {
    throw new NotFoundError("User");
  }
} catch (err) {
  if (err instanceof NotFoundError) {
    return res.status(404).json({ error: err.message });
  }
  throw err; // Re-throw unexpected errors
}
```

**Programmer Errors (bugs, should crash):**

```typescript
// These indicate bugs in the code
- TypeError (accessing property of undefined)
- ReferenceError (using undefined variable)
- Failed assertions
- Invalid arguments to internal functions
- State that "should never happen"

// Don't catch - fix the code
// Bad: hiding the bug
try {
  processItem(items[0].data);
} catch {
  return null;
}

// Good: fix the bug
if (items.length === 0 || !items[0].data) {
  throw new ValidationError("No items to process");
}
processItem(items[0].data);
```

**Decision guide:**

| Question | Operational | Programmer |
|----------|-------------|------------|
| Can user cause this? | Yes | No |
| Expected in production? | Yes | No |
| Can code recover? | Yes | No |
| Indicates a bug? | No | Yes |

Let programmer errors crash the process - trying to recover leaves the app in an unknown state.
