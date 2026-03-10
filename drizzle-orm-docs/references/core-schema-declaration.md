---
name: core-schema-declaration
description: Defining SQL schema in TypeScript with Drizzle ORM tables, columns, and naming conventions
---

# Schema Declaration

Define database tables as TypeScript objects. Drizzle supports PostgreSQL, MySQL, SQLite, MSSQL, SingleStore, and CockroachDB.

## Table Definition Styles

Three equivalent ways to define tables:

```typescript
// 1. Named imports
import { integer, pgTable, varchar } from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  name: varchar().notNull(),
  email: varchar().notNull().unique(),
});

// 2. Callback style (avoids circular references)
import { pgTable } from "drizzle-orm/pg-core";

export const users = pgTable("users", (t) => ({
  id: t.integer().primaryKey().generatedAlwaysAsIdentity(),
  name: t.varchar().notNull(),
  email: t.varchar().notNull().unique(),
}));

// 3. Wildcard import
import * as p from "drizzle-orm/pg-core";

export const users = p.pgTable("users", {
  id: p.integer().primaryKey().generatedAlwaysAsIdentity(),
  name: p.varchar().notNull(),
  email: p.varchar().notNull().unique(),
});
```

## Column Naming & Casing

By default, the TypeScript key becomes the database column name. Use an alias to decouple them:

```typescript
export const users = pgTable('users', {
  id: integer(),
  firstName: varchar('first_name'), // TS: firstName, DB: first_name
});
```

Automatic casing via the `drizzle()` config:

```typescript
import { drizzle } from "drizzle-orm/node-postgres";

const db = drizzle({ connection: process.env.DATABASE_URL, casing: 'snake_case' });

// Now camelCase keys auto-map to snake_case columns
export const users = pgTable('users', {
  id: integer(),
  firstName: varchar(), // DB column: first_name
});
```

## Reusable Column Patterns

```typescript
const timestamps = {
  updatedAt: timestamp(),
  createdAt: timestamp().defaultNow().notNull(),
  deletedAt: timestamp(),
};

export const users = pgTable('users', {
  id: integer(),
  ...timestamps,
});

export const posts = pgTable('posts', {
  id: integer(),
  ...timestamps,
});
```

## Schema Organization

Single file or multi-file schemas:

```typescript
// drizzle.config.ts — single file
export default defineConfig({
  dialect: 'postgresql',
  schema: './src/db/schema.ts',
});

// drizzle.config.ts — directory of schema files
export default defineConfig({
  dialect: 'postgresql',
  schema: './src/db/schema',
});
```

## Database-Specific Table Functions

| Database | Import | Table Function |
|----------|--------|----------------|
| PostgreSQL | `drizzle-orm/pg-core` | `pgTable` |
| MySQL | `drizzle-orm/mysql-core` | `mysqlTable` |
| SQLite | `drizzle-orm/sqlite-core` | `sqliteTable` |
| MSSQL | `drizzle-orm/mssql-core` | `mssqlTable` |
| SingleStore | `drizzle-orm/singlestore-core` | `singlestoreTable` |
| CockroachDB | `drizzle-orm/cockroach-core` | `cockroachTable` |

## Complete PostgreSQL Example

```typescript
import { AnyPgColumn, pgEnum, pgTable as table } from "drizzle-orm/pg-core";
import * as t from "drizzle-orm/pg-core";

export const rolesEnum = pgEnum("roles", ["guest", "user", "admin"]);

export const users = table(
  "users",
  {
    id: t.integer().primaryKey().generatedAlwaysAsIdentity(),
    firstName: t.varchar("first_name", { length: 256 }),
    lastName: t.varchar("last_name", { length: 256 }),
    email: t.varchar().notNull(),
    invitee: t.integer().references((): AnyPgColumn => users.id),
    role: rolesEnum().default("guest"),
  },
  (table) => [t.uniqueIndex("email_idx").on(table.email)]
);

export const posts = table(
  "posts",
  {
    id: t.integer().primaryKey().generatedAlwaysAsIdentity(),
    slug: t.varchar().$default(() => generateUniqueString(16)),
    title: t.varchar({ length: 256 }),
    ownerId: t.integer("owner_id").references(() => users.id),
  },
  (table) => [
    t.uniqueIndex("slug_idx").on(table.slug),
    t.index("title_idx").on(table.title),
  ]
);
```

## Complete MySQL Example

```typescript
import { mysqlTable as table } from "drizzle-orm/mysql-core";
import * as t from "drizzle-orm/mysql-core";

export const users = table("users", {
  id: t.int().primaryKey().autoincrement(),
  firstName: t.varchar("first_name", { length: 256 }),
  email: t.varchar({ length: 256 }).notNull(),
  role: t.mysqlEnum(["guest", "user", "admin"]).default("guest"),
});
```

## Complete SQLite Example

```typescript
import { sqliteTable as table } from "drizzle-orm/sqlite-core";
import * as t from "drizzle-orm/sqlite-core";

export const users = table("users", {
  id: t.int().primaryKey({ autoIncrement: true }),
  firstName: t.text("first_name"),
  email: t.text().notNull(),
  role: t.text().$type<"guest" | "user" | "admin">().default("guest"),
});
```

<!--
Source references:
- https://orm.drizzle.team/docs/sql-schema-declaration
-->
