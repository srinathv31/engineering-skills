---
title: parse vs safeParse
impact: CRITICAL
impactDescription: prevents unhandled exceptions in request handlers
tags: zod, validation, error-handling, request-handlers
---

## parse vs safeParse

Use `.safeParse()` when handling user input in request handlers to avoid unhandled exceptions. Use `.parse()` only when a thrown error is acceptable (e.g., configuration loading at startup).

**Incorrect (parse throws in request handler):**

```typescript
app.post("/users", async (req, res) => {
  // Throws ZodError on invalid input - crashes handler
  const data = UserSchema.parse(req.body);
  const user = await createUser(data);
  res.json(user);
});

// Without proper error middleware, this crashes the server
```

**Correct (safeParse for controlled error handling):**

```typescript
app.post("/users", async (req, res) => {
  const result = UserSchema.safeParse(req.body);

  if (!result.success) {
    // Zod v4: use z.flattenError() instead of .flatten()
    const { fieldErrors } = z.flattenError(result.error);
    return res.status(400).json({
      message: "Validation failed",
      errors: fieldErrors,
    });
  }

  const user = await createUser(result.data);
  res.json(user);
});
```

**When parse() is appropriate:**

```typescript
// Startup config - should crash if invalid
const config = ConfigSchema.parse(process.env);

// Internal data that should always be valid
function processInternalEvent(event: unknown) {
  const validated = InternalEventSchema.parse(event);
  // If this throws, it's a programmer error
}
```

**safeParse return types:**

```typescript
// Success case
{ success: true, data: T }

// Failure case
{ success: false, error: ZodError }
```

**Custom error messages (Zod v4):**

```typescript
// Use `error` param for custom messages (replaces `message`)
const schema = z.object({
  email: z.email({ error: "Please enter a valid email" }),
  age: z.number({ error: (issue) => `Age error: ${issue.input}` }),
});
```

Use `safeParse()` for external input, `parse()` for trusted internal data or startup configuration.
