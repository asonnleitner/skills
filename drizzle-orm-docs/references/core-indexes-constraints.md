---
name: core-indexes-constraints
description: Indexes, constraints, primary keys, foreign keys, and check constraints in Drizzle ORM
---

# Indexes & Constraints

## Primary Key

```typescript
// Single column
const table = pgTable('table', {
  id: integer('id').primaryKey(),
});

// Composite primary key
import { primaryKey } from "drizzle-orm/pg-core";

const table = pgTable('table', {
  id: integer('id'),
  name: varchar('name'),
}, (table) => [primaryKey({ columns: [table.id, table.name] })]);
```

## Identity / Auto-Increment

```typescript
// PostgreSQL — GENERATED ALWAYS AS IDENTITY
const table = pgTable('table', {
  id: integer('id').primaryKey().generatedAlwaysAsIdentity(),
});

// With options
const table = pgTable('table', {
  id: integer('id').primaryKey().generatedAlwaysAsIdentity({ startWith: 100 }),
});

// MySQL — AUTO_INCREMENT
const table = mysqlTable('table', {
  id: int('id').primaryKey().autoincrement(),
});

// SQLite — autoIncrement
const table = sqliteTable('table', {
  id: integer('id').primaryKey({ autoIncrement: true }),
});
```

## Not Null & Default

```typescript
import { sql } from "drizzle-orm";

const table = pgTable('table', {
  name: varchar('name').notNull(),
  age: integer('age').default(0),
  uuid: uuid('uuid').defaultRandom(),
  createdAt: timestamp('created_at').default(sql`now()`),
});
```

## Unique Constraint

```typescript
// Inline
const table = pgTable('table', {
  email: varchar('email').unique(),
});

// As unique index (third argument)
import { uniqueIndex } from "drizzle-orm/pg-core";

const table = pgTable('table', {
  email: varchar('email'),
}, (table) => [uniqueIndex('email_idx').on(table.email)]);
```

## Foreign Key

```typescript
// Inline reference
const posts = pgTable('posts', {
  userId: integer('user_id').references(() => users.id),
});

// With actions
const posts = pgTable('posts', {
  userId: integer('user_id').references(() => users.id, {
    onDelete: 'cascade',
    onUpdate: 'no action',
  }),
});

// Multi-column foreign key
import { foreignKey } from "drizzle-orm/pg-core";

const posts = pgTable('posts', {
  userId: integer('user_id'),
}, (table) => [
  foreignKey({
    columns: [table.userId],
    foreignColumns: [users.id],
  }),
]);
```

Self-referencing foreign key:

```typescript
import { AnyPgColumn } from "drizzle-orm/pg-core";

const users = pgTable('users', {
  id: integer('id').primaryKey(),
  parentId: integer('parent_id').references((): AnyPgColumn => users.id),
});
```

## Check Constraint

```typescript
import { check } from "drizzle-orm/pg-core";
import { sql } from "drizzle-orm";

const table = pgTable('table', {
  id: integer('id').primaryKey(),
  age: integer('age'),
}, (table) => [
  check('age_check', sql`${table.age} > 0`),
]);
```

## Indexes

```typescript
import { index, uniqueIndex } from "drizzle-orm/pg-core";

const table = pgTable('table', {
  id: integer('id').primaryKey(),
  name: varchar('name'),
  email: varchar('email'),
}, (table) => [
  index('name_idx').on(table.name),
  uniqueIndex('email_idx').on(table.email),
  // Multi-column index
  index('name_email_idx').on(table.name, table.email),
]);
```

<!--
Source references:
- https://orm.drizzle.team/docs/indexes-constraints
-->
