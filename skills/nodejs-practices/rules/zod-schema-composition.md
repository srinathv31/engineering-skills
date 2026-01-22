---
title: Schema Composition
impact: HIGH
impactDescription: eliminates field definition duplication and ensures consistent validation
tags: zod, validation, schema-design, composition
---

## Schema Composition

Use Zod's composition methods to avoid duplicating field definitions across schemas.

**Incorrect (duplicated definitions):**

```typescript
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
  password: z.string().min(8),
});

const UpdateUserSchema = z.object({
  email: z.string().email().optional(),
  name: z.string().min(1).optional(),
  // Duplicated field definitions!
  // If validation rules change, must update multiple places
});

const UserResponseSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),  // Duplicated again
  name: z.string().min(1),    // And again
  createdAt: z.date(),
});
```

**Correct (composed schemas):**

```typescript
// Base schema with shared fields (Zod v4 top-level validators)
const UserBaseSchema = z.object({
  email: z.email(),  // z.string().email() → z.email() in v4
  name: z.string().min(1),
});

// Extend for creation (adds fields)
const CreateUserSchema = UserBaseSchema.extend({
  password: z.string().min(8),
});

// Partial for updates (all optional)
const UpdateUserSchema = UserBaseSchema.partial();

// Extend for response (adds id, timestamps)
const UserResponseSchema = UserBaseSchema.extend({
  id: z.uuid(),  // z.string().uuid() → z.uuid() in v4
  createdAt: z.date(),
  updatedAt: z.date(),
});
```

**Composition methods (Zod v4):**

```typescript
// extend() - add new fields (preferred method)
const Extended = Base.extend({ newField: z.string() });

// Spread syntax - alternative to deprecated merge()
const Combined = z.object({ ...SchemaA.shape, ...SchemaB.shape });

// pick() - select specific fields
const Picked = Base.pick({ email: true, name: true });

// omit() - exclude specific fields
const Omitted = Base.omit({ password: true });

// partial() - make all fields optional
const Partial = Base.partial();

// required() - make all fields required
const Required = Base.required();

// z.strictObject() - reject unknown keys (replaces .strict())
const Strict = z.strictObject({ email: z.email() });

// z.looseObject() - allow unknown keys (replaces .passthrough())
const Loose = z.looseObject({ email: z.email() });
```

**Partial with specific fields:**

```typescript
// Only make specific fields optional
const PartialUpdate = Base.partial({
  name: true,
  email: true,
});
```

Schema composition ensures validation rules stay consistent and reduces maintenance overhead. Note: In Zod v4, `.merge()` and `.deepPartial()` are deprecated. Use spread syntax with `.shape` for merging and handle nested partials manually.
