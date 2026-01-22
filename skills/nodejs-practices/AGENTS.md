# Node.js Practices - Complete Guide

> This document is mainly for agents and LLMs. For the overview and quick reference, see `SKILL.md`.

## Abstract

This guide provides runtime-focused patterns for Node.js applications covering validation with Zod, async operation patterns, and error handling strategies. These patterns complement compile-time TypeScript type safety with runtime validation and error management.

## Table of Contents

1. [parse vs safeParse](#1-parse-vs-safeparse)
2. [Zod Error Formatting](#2-zod-error-formatting)
3. [Schema Composition](#3-schema-composition)
4. [Transform and Coerce](#4-transform-and-coerce)
5. [Promise.all vs Promise.allSettled](#5-promiseall-vs-promiseallsettled)
6. [Unhandled Rejections](#6-unhandled-rejections)
7. [Sequential vs Concurrent](#7-sequential-vs-concurrent)
8. [Async in Callbacks](#8-async-in-callbacks)
9. [Operational vs Programmer Errors](#9-operational-vs-programmer-errors)
10. [Custom Error Classes](#10-custom-error-classes)
11. [Error Propagation and Wrapping](#11-error-propagation-and-wrapping)

---

## 1. parse vs safeParse

**Impact: CRITICAL**

Use `.safeParse()` for user input in request handlers; use `.parse()` only when thrown errors are acceptable.

**Incorrect:**

```typescript
app.post("/users", async (req, res) => {
  const data = UserSchema.parse(req.body); // Throws on invalid input
  const user = await createUser(data);
  res.json(user);
});
```

**Correct:**

```typescript
app.post("/users", async (req, res) => {
  const result = UserSchema.safeParse(req.body);
  if (!result.success) {
    // Zod v4: use z.flattenError() instead of .flatten()
    return res.status(400).json({ errors: z.flattenError(result.error) });
  }
  const user = await createUser(result.data);
  res.json(user);
});

// parse() is fine for startup config
const config = ConfigSchema.parse(process.env);
```

---

## 2. Zod Error Formatting

**Impact: HIGH**

Never expose raw ZodError objects. Format errors for API consumers.

**Incorrect:**

```typescript
if (!result.success) {
  res.status(400).json(result.error); // Exposes internal structure
}
```

**Correct:**

```typescript
if (!result.success) {
  // Zod v4: use top-level z.flattenError()
  const { fieldErrors } = z.flattenError(result.error);
  res.status(400).json({ message: "Validation failed", errors: fieldErrors });
}
```

**Options (Zod v4):** `z.flattenError()` for flat structure, `z.treeifyError()` for nested, `z.prettifyError()` for human-readable, `.issues` for custom.

---

## 3. Schema Composition

**Impact: HIGH**

Use composition methods to avoid duplicating field definitions.

**Incorrect:**

```typescript
const CreateUserSchema = z.object({ email: z.email(), name: z.string() });
const UpdateUserSchema = z.object({ email: z.email().optional(), name: z.string().optional() }); // Duplicated!
```

**Correct:**

```typescript
// Zod v4: use z.email(), z.uuid() as top-level validators
const UserBaseSchema = z.object({ email: z.email(), name: z.string() });
const CreateUserSchema = UserBaseSchema.extend({ password: z.string().min(8) });
const UpdateUserSchema = UserBaseSchema.partial();
const PublicUserSchema = UserBaseSchema.omit({ email: true });
// Use spread for merging: z.object({ ...SchemaA.shape, ...SchemaB.shape })
```

**Methods (Zod v4):** `extend()`, `pick()`, `omit()`, `partial()`, `required()`, spread with `.shape` (replaces deprecated `merge()`/`deepPartial()`)

---

## 4. Transform and Coerce

**Impact: MEDIUM**

Use built-in transforms instead of manual conversion.

**Incorrect:**

```typescript
const data = schema.parse(input);
const age = parseInt(data.age, 10); // Manual conversion
```

**Correct:**

```typescript
const schema = z.object({
  age: z.coerce.number().int().positive(),
  date: z.coerce.date(),
  tags: z.string().transform(s => s.split(",").map(t => t.trim())),
});
```

**Options:** `z.coerce.*` for built-in coercion, `.transform()` for custom, `.refine()` for validation only. **Zod v4:** Use `.prefault()` for pre-parse defaults (input type), `.default()` for post-parse defaults (output type).

---

## 5. Promise.all vs Promise.allSettled

**Impact: CRITICAL**

Choose based on failure semantics: `Promise.all` fails fast, `Promise.allSettled` completes all.

**Incorrect:**

```typescript
// Need all results even if some fail - wrong method
const results = await Promise.all(userIds.map(id => fetchUser(id))); // Loses successes on any failure

// Any failure should abort - wrong method
const results = await Promise.allSettled(steps.map(s => processStep(s))); // Continues despite failures
```

**Correct:**

```typescript
// Need all results regardless of failures
const results = await Promise.allSettled(userIds.map(id => fetchUser(id)));
const data = results.map((r, i) => ({
  userId: userIds[i],
  data: r.status === "fulfilled" ? r.value : null,
  error: r.status === "rejected" ? r.reason : null,
}));

// All must succeed (transactions)
const results = await Promise.all(steps.map(s => processStep(s)));
```

---

## 6. Unhandled Rejections

**Impact: CRITICAL**

Always handle promise rejections. Unhandled rejections crash Node.js v15+.

**Incorrect:**

```typescript
logAnalytics(event); // No .catch(), no await - crashes if fails
```

**Correct:**

```typescript
// Catch explicitly
logAnalytics(event).catch(err => console.error("Analytics failed:", err));

// Or use void + catch for intentional fire-and-forget
void logAnalytics(event).catch(err => console.error("Analytics failed:", err));

// Or await with try/catch
try {
  await logAnalytics(event);
} catch (err) {
  console.error("Analytics failed:", err);
}
```

Enable `@typescript-eslint/no-floating-promises` to catch these at lint time.

---

## 7. Sequential vs Concurrent

**Impact: HIGH**

Run independent operations concurrently with `Promise.all`.

**Incorrect:**

```typescript
const user = await fetchUser(userId);         // 100ms
const posts = await fetchPosts(userId);       // 100ms
const notifications = await fetchNotifications(userId); // 100ms
// Total: ~300ms
```

**Correct:**

```typescript
const [user, posts, notifications] = await Promise.all([
  fetchUser(userId),
  fetchPosts(userId),
  fetchNotifications(userId),
]);
// Total: ~100ms

// Sequential only when operations depend on each other
const order = await fetchOrder(orderId);
const payment = await chargeCustomer(order.customerId, order.total);
```

---

## 8. Async in Callbacks

**Impact: HIGH**

Never use async with `.forEach()`. Use `for...of` or `Promise.all` with `.map()`.

**Incorrect:**

```typescript
items.forEach(async (item) => {
  await processItem(item); // Not awaited by forEach!
});
console.log("Done"); // Runs immediately
```

**Correct:**

```typescript
// Sequential
for (const item of items) {
  await processItem(item);
}

// Concurrent
await Promise.all(items.map(item => processItem(item)));

// Async filtering
const results = await Promise.all(items.map(async item => ({ item, valid: await isValid(item) })));
return results.filter(r => r.valid).map(r => r.item);
```

Same applies to `.filter()`, `.some()`, `.every()`, `.reduce()` - they don't await async callbacks.

---

## 9. Operational vs Programmer Errors

**Impact: CRITICAL**

Handle operational errors gracefully; let programmer errors crash.

**Incorrect:**

```typescript
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message }); // All errors same
});

try {
  const name = items[0].name.toUpperCase();
} catch {
  return null; // Hides bug
}
```

**Correct:**

```typescript
class NotFoundError extends Error {
  readonly isOperational = true;
  readonly statusCode = 404;
}

app.use((err, req, res, next) => {
  if (err.isOperational) {
    return res.status(err.statusCode).json({ error: err.message });
  }
  logger.fatal("Programmer error", err);
  process.exit(1);
});

// Fix bugs, don't catch them
if (items.length === 0) throw new NotFoundError("No items");
const name = items[0].name.toUpperCase();
```

**Operational:** File not found, timeout, invalid input, constraint violations
**Programmer:** TypeError, ReferenceError, assertion failures

---

## 10. Custom Error Classes

**Impact: HIGH**

Create typed error classes with status codes and metadata.

**Incorrect:**

```typescript
throw new Error("User not found");
if (err.message.includes("not found")) { /* brittle */ }
```

**Correct:**

```typescript
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
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string) {
    super(`${resource} not found: ${id}`, 404, "NOT_FOUND", { resource, id });
  }
}

// Usage
throw new NotFoundError("User", userId);

if (err instanceof NotFoundError) {
  res.status(err.statusCode).json({ code: err.code, message: err.message });
}
```

---

## 11. Error Propagation and Wrapping

**Impact: MEDIUM**

Preserve error context with the `cause` option when rethrowing.

**Incorrect:**

```typescript
try {
  return await db.users.findById(id);
} catch (err) {
  throw new Error("Failed to get user"); // Original error lost
}
```

**Correct:**

```typescript
try {
  return await db.users.findById(id);
} catch (err) {
  throw new AppError("Failed to get user", 500, "DATABASE_ERROR", { userId: id }, { cause: err });
}

// AppError with cause support
class AppError extends Error {
  constructor(
    message: string,
    readonly statusCode: number,
    readonly code: string,
    readonly metadata?: Record<string, unknown>,
    options?: ErrorOptions
  ) {
    super(message, options);
  }
}

// Log cause chain
function logError(err: Error, depth = 0) {
  console.error(`${"  ".repeat(depth)}${err.name}: ${err.message}`);
  if (err.cause instanceof Error) logError(err.cause, depth + 1);
}
```
