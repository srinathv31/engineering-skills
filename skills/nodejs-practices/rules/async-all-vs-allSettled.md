---
title: Promise.all vs Promise.allSettled
impact: CRITICAL
impactDescription: ensures correct failure semantics for concurrent operations
tags: async, promises, concurrency, error-handling
---

## Promise.all vs Promise.allSettled

Choose the correct Promise combinator based on your failure semantics. Using the wrong one can lose data or cause inconsistent state.

**Incorrect (wrong combinator for use case):**

```typescript
// When you need all results even if some fail - WRONG
async function sendNotifications(userIds: string[]) {
  // If one notification fails, all successful ones are lost
  const results = await Promise.all(
    userIds.map(id => sendNotification(id))
  );
  return results;
}

// When any failure should abort - WRONG
async function processTransaction(steps: Step[]) {
  // Continues despite failures - may leave inconsistent state
  const results = await Promise.allSettled(
    steps.map(step => executeStep(step))
  );
  // Some steps failed but we continued anyway
}
```

**Correct (matching combinator to semantics):**

```typescript
// Need all results regardless of individual failures
async function sendNotifications(userIds: string[]) {
  const results = await Promise.allSettled(
    userIds.map(id => sendNotification(id))
  );

  const successful = results
    .filter((r): r is PromiseFulfilledResult<void> => r.status === "fulfilled")
    .length;
  const failed = results
    .filter((r): r is PromiseRejectedResult => r.status === "rejected");

  console.log(`Sent ${successful}/${userIds.length} notifications`);
  return { successful, failed };
}

// All must succeed or entire operation fails
async function processTransaction(steps: Step[]) {
  // Fails fast - if any step fails, throws immediately
  const results = await Promise.all(
    steps.map(step => executeStep(step))
  );
  // Only reaches here if ALL steps succeeded
  return results;
}
```

**When to use which:**

| Use Case | Method |
|----------|--------|
| All must succeed (transactions, dependent ops) | `Promise.all` |
| Need all results (batch ops, notifications) | `Promise.allSettled` |
| First to succeed wins | `Promise.any` |
| First to complete wins | `Promise.race` |

**Processing allSettled results:**

```typescript
const results = await Promise.allSettled(promises);

for (const result of results) {
  if (result.status === "fulfilled") {
    console.log("Success:", result.value);
  } else {
    console.log("Failed:", result.reason);
  }
}

// Separate successes and failures
const successes = results
  .filter((r): r is PromiseFulfilledResult<T> => r.status === "fulfilled")
  .map(r => r.value);

const failures = results
  .filter((r): r is PromiseRejectedResult => r.status === "rejected")
  .map(r => r.reason);
```

Choose based on whether partial success is acceptable or the entire operation must be atomic.
