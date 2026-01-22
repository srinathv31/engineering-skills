---
title: Sequential vs Concurrent Execution
impact: HIGH
impactDescription: improves performance by running independent operations concurrently
tags: async, promises, performance, concurrency
---

## Sequential vs Concurrent Execution

Run independent operations concurrently with `Promise.all`. Use sequential `await` only for operations that depend on each other.

**Incorrect (sequential when operations are independent):**

```typescript
// Each await blocks the next - unnecessarily slow
async function getUserDashboard(userId: string) {
  const user = await fetchUser(userId);         // 100ms
  const posts = await fetchPosts(userId);       // 100ms
  const notifications = await fetchNotifications(userId); // 100ms
  // Total: ~300ms (sequential)
  return { user, posts, notifications };
}

// Same problem with loops
async function fetchAllUsers(ids: string[]) {
  const users = [];
  for (const id of ids) {
    const user = await fetchUser(id);  // Each one waits for the previous
    users.push(user);
  }
  return users;
}
```

**Correct (concurrent for independent operations):**

```typescript
// All fetch operations run in parallel
async function getUserDashboard(userId: string) {
  const [user, posts, notifications] = await Promise.all([
    fetchUser(userId),
    fetchPosts(userId),
    fetchNotifications(userId),
  ]);
  // Total: ~100ms (concurrent)
  return { user, posts, notifications };
}

// Concurrent loop processing
async function fetchAllUsers(ids: string[]) {
  const users = await Promise.all(
    ids.map(id => fetchUser(id))
  );
  return users;
}
```

**Sequential when operations depend on each other:**

```typescript
// Each step needs the result of the previous one
async function processOrder(orderId: string) {
  // Must be sequential - each step needs previous result
  const order = await fetchOrder(orderId);
  const payment = await chargeCustomer(order.customerId, order.total);
  const shipment = await createShipment(order, payment.transactionId);
  return { order, payment, shipment };
}

// Mixed: some sequential, some concurrent
async function processUserData(userId: string) {
  // First, fetch user (needed for other operations)
  const user = await fetchUser(userId);

  // Then, fetch user-dependent data concurrently
  const [orders, preferences] = await Promise.all([
    fetchOrders(user.id),
    fetchPreferences(user.id),
  ]);

  return { user, orders, preferences };
}
```

**Controlled concurrency for large batches:**

```typescript
// Limit concurrent operations to avoid overwhelming resources
async function fetchUsersWithLimit(ids: string[], limit = 5) {
  const results: User[] = [];

  for (let i = 0; i < ids.length; i += limit) {
    const batch = ids.slice(i, i + limit);
    const batchResults = await Promise.all(
      batch.map(id => fetchUser(id))
    );
    results.push(...batchResults);
  }

  return results;
}
```

Ask: "Does this operation depend on the result of the previous one?" If no, run them concurrently.
