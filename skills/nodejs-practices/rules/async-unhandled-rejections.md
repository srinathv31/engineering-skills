---
title: Unhandled Rejections
impact: CRITICAL
impactDescription: prevents process crashes from unhandled promise rejections
tags: async, promises, error-handling, unhandled-rejection
---

## Unhandled Rejections

Always handle promise rejections. Unhandled rejections crash Node.js processes in v15+ and will terminate the process by default.

**Incorrect (fire and forget without handling):**

```typescript
// No error handling - crashes process if sendToAnalytics fails
async function logAnalytics(event: Event) {
  await sendToAnalytics(event);
}
logAnalytics(event); // Not awaited, no .catch()

// Promise returned but caller might ignore it
function startBackgroundJob() {
  return doExpensiveWork(); // Unhandled if caller doesn't await
}
startBackgroundJob(); // Floating promise

// Forgetting to await in async function
async function processRequest(req: Request) {
  validateRequest(req); // Returns promise but not awaited
  // Validation errors won't be caught
}
```

**Correct (explicit error handling):**

```typescript
// Option 1: Catch errors with .catch()
logAnalytics(event).catch(err => {
  console.error("Analytics failed:", err);
  // Don't rethrow - this is fire-and-forget
});

// Option 2: void prefix + catch (marks intentional fire-and-forget)
void logAnalytics(event).catch(err => {
  console.error("Analytics failed:", err);
});

// Option 3: await with try/catch
try {
  await logAnalytics(event);
} catch (err) {
  console.error("Analytics failed:", err);
}

// Background jobs should handle their own errors
function startBackgroundJob() {
  doExpensiveWork()
    .then(result => {
      console.log("Background job completed:", result);
    })
    .catch(err => {
      console.error("Background job failed:", err);
    });
  // No return - truly fire-and-forget with error handling
}
```

**ESLint rule to catch floating promises:**

```json
{
  "rules": {
    "@typescript-eslint/no-floating-promises": "error"
  }
}
```

**Global unhandled rejection handler (last resort):**

```typescript
process.on("unhandledRejection", (reason, promise) => {
  console.error("Unhandled Rejection at:", promise, "reason:", reason);
  // Log and exit - don't swallow
  process.exit(1);
});
```

**Patterns for intentional fire-and-forget:**

```typescript
// Document the intent
/** @fire-and-forget Errors are logged internally */
function scheduleCleanup() {
  void performCleanup().catch(logError);
}

// Utility for safe fire-and-forget
function fireAndForget(promise: Promise<unknown>, context: string) {
  promise.catch(err => {
    console.error(`Fire-and-forget failed [${context}]:`, err);
  });
}

fireAndForget(sendAnalytics(event), "analytics");
```

Every promise must either be awaited, returned, or have a `.catch()` handler attached.
