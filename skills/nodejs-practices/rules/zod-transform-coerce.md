---
title: Transform and Coerce
impact: MEDIUM
impactDescription: leverages built-in type conversion for cleaner validation pipelines
tags: zod, validation, transform, coercion, type-conversion
---

## Transform and Coerce

Use Zod's built-in transforms and coercion instead of manual type conversion after validation.

**Incorrect (manual conversion):**

```typescript
const schema = z.object({
  age: z.string(),
  date: z.string(),
  price: z.string(),
});

// Manual conversion after validation
const data = schema.parse(input);
const age = parseInt(data.age, 10);
const date = new Date(data.date);
const price = parseFloat(data.price);

// No validation that conversion succeeded
if (isNaN(age)) { /* handle error */ }
```

**Correct (built-in coercion):**

```typescript
const schema = z.object({
  // Coerce from string to number with validation
  age: z.coerce.number().int().positive(),
  // Coerce from string to Date
  date: z.coerce.date(),
  // Coerce and validate
  price: z.coerce.number().positive(),
});

const data = schema.parse(input);
// data.age is number, data.date is Date, data.price is number
// Invalid values fail during parsing with proper error messages
```

**Transform for custom logic:**

```typescript
const schema = z.object({
  // Split comma-separated string into array
  tags: z.string().transform(s => s.split(",").map(t => t.trim())),

  // Normalize email (use z.email() in v4)
  email: z.email().transform(s => s.toLowerCase()),

  // Parse JSON string (Zod v4 uses err.issues array)
  metadata: z.string().transform((s, ctx) => {
    try {
      return JSON.parse(s);
    } catch {
      ctx.issues.push({
        code: "custom",
        message: "Invalid JSON",
        input: s,
        path: [],
      });
      return z.NEVER;
    }
  }),
});
```

**Coerce vs Transform vs Refine:**

```typescript
// z.coerce.* - built-in type coercion
z.coerce.number()   // Calls Number(input)
z.coerce.boolean()  // Calls Boolean(input)
z.coerce.date()     // Calls new Date(input)
z.coerce.string()   // Calls String(input)

// .transform() - custom transformation, changes output type
z.string().transform(s => s.length)  // string -> number

// .refine() - custom validation, same output type
z.string().refine(s => s.length > 0, "Required")

// Chain transform with further validation
z.string()
  .transform(s => parseInt(s, 10))
  .pipe(z.number().positive())
```

**Defaults (Zod v4):**

```typescript
// .default() - applied after parsing, expects OUTPUT type
const WithDefault = z.coerce.number().default(0);  // default is number, not string

// .prefault() - applied before parsing, expects INPUT type (Zod v4)
const WithPrefault = z.coerce.number().prefault("0");  // prefault is string
```

**Preprocessing:**

```typescript
// preprocess() runs before parsing
const trimmedString = z.preprocess(
  (val) => (typeof val === "string" ? val.trim() : val),
  z.string()
);
```

Use coercion for standard type conversions, transforms for custom logic, and refine for additional validation. In Zod v4, use `.prefault()` for pre-parse defaults and `.default()` for post-parse defaults.
