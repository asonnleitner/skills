---
name: features-schemas
description: Database schema namespacing in Drizzle ORM for PostgreSQL, MySQL, and others
---

# Database Schemas

Create tables under a named schema (not the default/public schema). Supported in PostgreSQL, MySQL, SingleStore, MSSQL, CockroachDB. Not supported in SQLite.

## PostgreSQL

```typescript
import { serial, text, pgSchema } from "drizzle-orm/pg-core";

export const mySchema = pgSchema("my_schema");

export const colors = mySchema.enum('colors', ['red', 'green', 'blue']);

export const users = mySchema.table('users', {
  id: serial('id').primaryKey(),
  name: text('name'),
  color: colors('color').default('red'),
});
```

Generates:
```sql
CREATE SCHEMA "my_schema";
CREATE TYPE "my_schema"."colors" AS ENUM ('red', 'green', 'blue');
CREATE TABLE "my_schema"."users" (
  "id" serial PRIMARY KEY,
  "name" text,
  "color" "my_schema"."colors" DEFAULT 'red'
);
```

## MySQL

```typescript
import { int, text, mysqlSchema } from "drizzle-orm/mysql-core";

export const mySchema = mysqlSchema("my_schema");
export const users = mySchema.table("users", {
  id: int("id").primaryKey().autoincrement(),
  name: text("name"),
});
```

## Other Dialects

- SingleStore: `singlestoreSchema()`
- MSSQL: `mssqlSchema()`
- CockroachDB: `cockroachSchema()`

All follow the same pattern: create schema, then use `schema.table()`, `schema.enum()`, `schema.sequence()` etc.

<!--
Source references:
- https://orm.drizzle.team/docs/schemas
-->
