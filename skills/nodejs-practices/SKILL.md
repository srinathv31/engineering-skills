---
name: nodejs-practices
description: Node.js runtime patterns for validation (Zod v4), async operations, and error handling. Apply when using Zod for validation, working with Promise combinators, or implementing error handling strategies. Triggers on schema validation, concurrent operations, async callbacks, or custom error classes.
license: MIT
metadata:
  author: community
  version: "1.1.0"
---

# Node.js Practices

Runtime-focused patterns for validation, async operations, and error handling in Node.js applications. Contains 11 rules across 3 categories complementing compile-time TypeScript patterns with runtime best practices.

## When to Apply

Reference these guidelines when:
- Using Zod for runtime validation (parse vs safeParse)
- Composing or extending Zod schemas
- Choosing between `Promise.all()` and `Promise.allSettled()`
- Running multiple async operations concurrently or sequentially
- Using async functions in callbacks or array methods
- Handling unhandled promise rejections
- Creating custom error classes
- Distinguishing operational vs programmer errors
- Wrapping or propagating errors through layers

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Validation (Zod) | CRITICAL-HIGH | `zod-` |
| 2 | Async Patterns | CRITICAL-HIGH | `async-` |
| 3 | Error Handling | CRITICAL-MEDIUM | `error-` |

## Quick Reference

### 1. Validation with Zod (CRITICAL-HIGH)

- `zod-parse-vs-safeParse` - Use safeParse in request handlers, parse only when errors should throw
- `zod-error-formatting` - Format Zod errors for API responses, never expose raw ZodError
- `zod-schema-composition` - Use extend/merge/pick/omit to compose schemas, avoid duplication
- `zod-transform-coerce` - Use built-in transforms and coercion instead of manual conversion

### 2. Async Patterns (CRITICAL-HIGH)

- `async-all-vs-allSettled` - Choose Promise method based on failure semantics needed
- `async-unhandled-rejections` - Always attach .catch() or use try/catch for async operations
- `async-sequential-vs-concurrent` - Use Promise.all for independent operations, await for dependent
- `async-await-in-callbacks` - Never use async with forEach, use for...of or Promise.all with map

### 3. Error Handling (CRITICAL-MEDIUM)

- `error-operational-vs-programmer` - Handle operational errors gracefully, crash on programmer errors
- `error-custom-classes` - Create typed error classes with status codes and metadata
- `error-propagation-wrapping` - Preserve cause chain when wrapping errors

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/zod-parse-vs-safeParse.md
rules/async-all-vs-allSettled.md
rules/error-operational-vs-programmer.md
```

Each rule file contains:
- Brief explanation of why it matters
- Incorrect code example with explanation
- Correct code example with explanation
- Additional context and variations

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
