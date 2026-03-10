---
name: core-indexes-constraints
description: Drizzle ORM indexes, constraints, foreign keys, and check constraints
---

# Indexes & Constraints

## Column Constraints

```ts
import { pgTable, integer, varchar, text, uuid, timestamp } from "drizzle-orm/pg-core";
import { sql } from "drizzle-orm";

export const users = pgTable("users", {
  // Default values
  age: integer().default(18),
  role: text().default("user"),
  id: uuid().defaultRandom(),                    // gen_random_uuid()
  createdAt: timestamp().defaultNow(),            // NOW()
  custom: integer().default(sql`abs(42)`),        // SQL expression

  // Not null
  name: varchar({ length: 255 }).notNull(),

  // Primary key
  id: integer().primaryKey(),

  // Unique
  email: varchar({ length: 255 }).unique(),
});
```

## Table-Level Constraints (3rd argument)

```ts
import { pgTable, integer, varchar, primaryKey, unique, check, index, uniqueIndex } from "drizzle-orm/pg-core";
import { sql } from "drizzle-orm";

export const table = pgTable("table", {
  id: integer(),
  name: varchar({ length: 255 }),
  email: varchar({ length: 255 }),
  age: integer(),
}, (t) => [
  // Composite primary key
  primaryKey({ columns: [t.id, t.name] }),

  // Named unique constraint
  unique("email_unique").on(t.email),

  // Composite unique
  unique().on(t.name, t.email),

  // Check constraint
  check("age_check", sql`${t.age} > 0`),

  // Index
  index("name_idx").on(t.name),

  // Unique index
  uniqueIndex("email_idx").on(t.email),

  // Partial index
  index("active_idx").on(t.name).where(sql`active = true`),

  // Expression index
  index("lower_name_idx").on(sql`lower(${t.name})`),
]);
```

## Foreign Keys

```ts
import { pgTable, integer, foreignKey } from "drizzle-orm/pg-core";

// Inline reference
export const posts = pgTable("posts", {
  authorId: integer("author_id").references(() => users.id),
});

// With actions
export const posts = pgTable("posts", {
  authorId: integer("author_id").references(() => users.id, {
    onDelete: "cascade",
    onUpdate: "no action",
  }),
});

// Self-reference (requires AnyPgColumn type)
import { type AnyPgColumn } from "drizzle-orm/pg-core";

export const categories = pgTable("categories", {
  id: integer().primaryKey(),
  parentId: integer("parent_id").references((): AnyPgColumn => categories.id),
});

// Table-level foreign key (composite)
export const table = pgTable("table", {
  col1: integer(),
  col2: integer(),
}, (t) => [
  foreignKey({
    columns: [t.col1, t.col2],
    foreignColumns: [otherTable.id1, otherTable.id2],
  }).onDelete("cascade"),
]);
```

### Foreign Key Actions

| Action | Description |
|--------|-------------|
| `"cascade"` | Delete/update child rows when parent changes |
| `"restrict"` | Prevent parent deletion if children exist |
| `"no action"` | Similar to restrict (checked at end of transaction) |
| `"set null"` | Set child FK to NULL |
| `"set default"` | Set child FK to default value |

## Views

```ts
// Regular view
export const userView = pgView("user_view").as((qb) =>
  qb.select({ id: users.id, name: users.name }).from(users)
);

// Materialized view (PG only)
export const matView = pgMaterializedView("mat_view").as((qb) =>
  qb.select().from(users)
);

// Query from view
await db.select().from(userView);

// Refresh materialized view
await db.refreshMaterializedView(matView);
```

<!--
Source references:
- https://orm.drizzle.team/docs/indexes-constraints
- https://orm.drizzle.team/docs/views
-->
