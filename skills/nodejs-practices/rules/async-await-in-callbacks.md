---
title: Async in Callbacks
impact: HIGH
impactDescription: prevents uncontrolled async execution in array methods
tags: async, promises, array-methods, forEach, callbacks
---

## Async in Callbacks

Never use async functions with `.forEach()` or callbacks that don't await the result. The callback returns a promise that is immediately discarded.

**Incorrect (async with forEach):**

```typescript
// forEach doesn't await - all promises fire immediately, untracked
async function processItems(items: Item[]) {
  items.forEach(async (item) => {
    await processItem(item); // Returns promise, but forEach ignores it
  });
  console.log("Done"); // Runs immediately, before any processing completes!
}

// Same problem with other callbacks
const results = items.map(async (item) => {
  return await transform(item);
}); // results is Promise<T>[], not T[]

// Async filter doesn't work as expected
const filtered = items.filter(async (item) => {
  return await isValid(item);  // Returns Promise, which is always truthy!
});
```

**Correct (for...of for sequential):**

```typescript
// Sequential processing with for...of
async function processItemsSequential(items: Item[]) {
  for (const item of items) {
    await processItem(item);
  }
  console.log("Done"); // Runs after all items are processed
}
```

**Correct (Promise.all for concurrent):**

```typescript
// Concurrent processing with Promise.all + map
async function processItemsConcurrent(items: Item[]) {
  await Promise.all(
    items.map(item => processItem(item))
  );
  console.log("Done"); // Runs after all items are processed
}

// Getting transformed results
const results = await Promise.all(
  items.map(item => transform(item))
);
// results is T[], not Promise<T>[]
```

**Async filtering pattern:**

```typescript
// Filter with async predicate
async function filterAsync<T>(
  items: T[],
  predicate: (item: T) => Promise<boolean>
): Promise<T[]> {
  const results = await Promise.all(
    items.map(async (item) => ({
      item,
      keep: await predicate(item),
    }))
  );
  return results.filter(r => r.keep).map(r => r.item);
}

// Usage
const validItems = await filterAsync(items, item => isValid(item));
```

**Async reduce pattern:**

```typescript
// Sequential reduce with async
async function sumAsync(items: Item[]): Promise<number> {
  let total = 0;
  for (const item of items) {
    total += await getValue(item);
  }
  return total;
}

// Or with reduce (sequential)
const total = await items.reduce(async (accPromise, item) => {
  const acc = await accPromise;
  const value = await getValue(item);
  return acc + value;
}, Promise.resolve(0));
```

**Methods that don't work with async callbacks:**
- `forEach` - ignores return value
- `filter` - Promise is truthy, all items pass
- `some` - Promise is truthy, returns true immediately
- `every` - Promise is truthy, returns true immediately
- `find` - Promise is truthy, returns first item
- `reduce` - works but needs careful handling

Use `for...of` for sequential or `Promise.all` with `map` for concurrent async operations.
