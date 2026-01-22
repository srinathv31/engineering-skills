---
title: Zod Error Formatting
impact: HIGH
impactDescription: ensures consistent, user-friendly API error responses
tags: zod, validation, api, error-formatting
---

## Zod Error Formatting

Never expose raw ZodError objects to API consumers. Format errors into a consistent, user-friendly structure.

**Incorrect (exposing raw ZodError):**

```typescript
const result = schema.safeParse(data);
if (!result.success) {
  // Exposes internal Zod structure with complex nested objects
  res.status(400).json(result.error);
}

// Raw ZodError has confusing structure for API consumers:
// { issues: [{ code: "invalid_type", ... }], ... }
```

**Correct (formatted errors):**

```typescript
const result = schema.safeParse(data);
if (!result.success) {
  // z.flattenError() provides clean structure (Zod v4)
  const { fieldErrors, formErrors } = z.flattenError(result.error);

  res.status(400).json({
    message: "Validation failed",
    errors: fieldErrors,
    // { email: ["Invalid email"], age: ["Expected number"] }
  });
}
```

**Error formatting methods (Zod v4):**

```typescript
// z.flattenError() - flat object with fieldErrors and formErrors
const flat = z.flattenError(error);
// {
//   formErrors: ["Root-level errors"],
//   fieldErrors: { email: ["Invalid email"] }
// }

// z.treeifyError() - nested structure matching schema shape
const tree = z.treeifyError(error);
// {
//   email: { _errors: ["Invalid email"] },
//   address: { city: { _errors: ["Required"] } }
// }

// z.prettifyError() - human-readable string output
const pretty = z.prettifyError(error);
// "email: Invalid email\nage: Expected number"

// issues - raw array for custom formatting
const issues = error.issues;
// [{ path: ["email"], message: "Invalid email", code: "invalid_string" }]
```

**Custom formatting example:**

```typescript
function formatZodError(error: ZodError): Record<string, string> {
  const errors: Record<string, string> = {};
  for (const issue of error.issues) {
    const path = issue.path.join(".");
    errors[path] = issue.message;
  }
  return errors;
}
```

Choose the formatting method that best matches your API's error response structure. Note: In Zod v4, formatting methods are top-level functions (`z.flattenError()`, `z.treeifyError()`, `z.prettifyError()`) rather than instance methods.
