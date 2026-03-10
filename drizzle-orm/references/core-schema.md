---
name: core-schema
description: Drizzle ORM schema declaration - tables, columns, and type inference across PostgreSQL, MySQL, and SQLite
---

# Schema Declaration

Drizzle schemas are defined in TypeScript and serve as the source of truth for queries (drizzle-orm) and migrations (drizzle-kit).

## Table Definition

Each dialect has its own table constructor. Import from the appropriate module:

```ts
// PostgreSQL
import { pgTable, integer, varchar, text, boolean, timestamp, uuid, serial, jsonb } from "drizzle-orm/pg-core";

// MySQL
import { mysqlTable, int, varchar, text, boolean, timestamp, serial } from "drizzle-orm/mysql-core";

// SQLite
import { sqliteTable, integer, text } from "drizzle-orm/sqlite-core";
```

### PostgreSQL Table

```ts
export const users = pgTable("users", {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  name: varchar({ length: 255 }).notNull(),
  email: varchar({ length: 255 }).notNull().unique(),
  age: integer(),
  role: text().$type<"admin" | "user">().default("user"),
  createdAt: timestamp({ withTimezone: true }).notNull().defaultNow(),
});
```

### Callback Syntax (alternative)

```ts
export const users = pgTable("users", (t) => ({
  id: t.integer().primaryKey().generatedAlwaysAsIdentity(),
  name: t.varchar({ length: 255 }).notNull(),
  email: t.varchar({ length: 255 }).notNull().unique(),
}));
```

### MySQL Table

```ts
export const users = mysqlTable("users", {
  id: int().primaryKey().autoincrement(),
  name: varchar({ length: 255 }).notNull(),
  email: varchar({ length: 255 }).notNull().unique(),
  createdAt: timestamp().notNull().defaultNow(),
});
```

### SQLite Table

```ts
export const users = sqliteTable("users", {
  id: integer().primaryKey({ autoIncrement: true }),
  name: text().notNull(),
  email: text().notNull().unique(),
  createdAt: integer({ mode: "timestamp" }).notNull().$defaultFn(() => new Date()),
});
```

## Column Aliases

Use column aliases when TypeScript key names differ from database column names:

```ts
export const users = pgTable("users", {
  id: integer(),
  firstName: varchar("first_name"),  // DB column is "first_name", TS key is "firstName"
});
```

## Schema Organization

### Single file

```ts
// drizzle.config.ts
export default defineConfig({
  dialect: "postgresql",
  schema: "./src/db/schema.ts",
});
```

### Multiple files

```ts
// drizzle.config.ts - reads all files in directory recursively
export default defineConfig({
  dialect: "postgresql",
  schema: "./src/db/schema",
});
```

## Type Inference

```ts
// Infer select type (what you get back from queries)
type User = typeof users.$inferSelect;

// Infer insert type (what you pass to insert)
type NewUser = typeof users.$inferInsert;
```

## Column Modifiers

| Modifier | Description |
|----------|-------------|
| `.notNull()` | NOT NULL constraint |
| `.default(value)` | Static default value |
| `.defaultNow()` | DEFAULT NOW() (timestamps) |
| `.$defaultFn(() => value)` | Runtime default (generated in JS, not DB) |
| `.$onUpdateFn(() => value)` | Runtime value on update |
| `.primaryKey()` | Primary key |
| `.unique()` | Unique constraint |
| `.references(() => table.col)` | Foreign key reference |
| `.$type<T>()` | Override inferred TypeScript type |
| `.generatedAlwaysAsIdentity()` | PG identity column (recommended over serial) |
| `.autoincrement()` | MySQL auto-increment |
| `.generatedAlwaysAs(sql)` | Generated/computed column |

## PostgreSQL Common Column Types

| Type | Import | Notes |
|------|--------|-------|
| `integer()` | `pg-core` | 4-byte integer |
| `bigint()` | `pg-core` | 8-byte integer, mode: "bigint" or "number" |
| `smallint()` | `pg-core` | 2-byte integer |
| `serial()` | `pg-core` | Auto-incrementing (prefer `generatedAlwaysAsIdentity`) |
| `varchar({ length })` | `pg-core` | Variable-length string |
| `text()` | `pg-core` | Unlimited text |
| `boolean()` | `pg-core` | true/false |
| `timestamp()` | `pg-core` | Options: `{ withTimezone: true, mode: "date" \| "string" }` |
| `date()` | `pg-core` | Options: `{ mode: "date" \| "string" }` |
| `uuid()` | `pg-core` | UUID, use `.defaultRandom()` for auto-gen |
| `jsonb()` | `pg-core` | JSONB, use `.$type<T>()` for typing |
| `json()` | `pg-core` | JSON |
| `numeric()` | `pg-core` | Arbitrary precision |
| `real()` | `pg-core` | 4-byte float |
| `doublePrecision()` | `pg-core` | 8-byte float |
| `pgEnum()` | `pg-core` | PostgreSQL enum type |

### PostgreSQL Enums

```ts
import { pgEnum } from "drizzle-orm/pg-core";

export const roleEnum = pgEnum("role", ["admin", "user", "guest"]);

export const users = pgTable("users", {
  id: integer().primaryKey(),
  role: roleEnum().default("user"),
});
```

## Schemas (PostgreSQL only)

```ts
import { pgSchema } from "drizzle-orm/pg-core";

export const mySchema = pgSchema("my_schema");

export const users = mySchema.table("users", {
  id: integer().primaryKey(),
  name: text(),
});
// Generates: "my_schema"."users"
```

<!--
Source references:
- https://orm.drizzle.team/docs/sql-schema-declaration
- https://orm.drizzle.team/docs/column-types/pg
- https://orm.drizzle.team/docs/column-types/mysql
- https://orm.drizzle.team/docs/column-types/sqlite
- https://orm.drizzle.team/docs/schemas
-->
