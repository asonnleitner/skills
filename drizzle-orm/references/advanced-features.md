---
name: advanced-features
description: Drizzle ORM advanced features - dynamic queries, batch API, read replicas, custom types, and RLS
---

# Advanced Features

## Dynamic Query Building

By default, query builder methods can only be called once. Use `.$dynamic()` to enable reusable query helpers:

```ts
import { type PgSelect } from "drizzle-orm/pg-core";

function withPagination<T extends PgSelect>(qb: T, page = 1, pageSize = 10) {
  return qb.limit(pageSize).offset((page - 1) * pageSize);
}

function withFilters<T extends PgSelect>(qb: T, role?: string) {
  if (role) return qb.where(eq(users.role, role));
  return qb;
}

// Usage
let query = db.select().from(users).$dynamic();
query = withFilters(query, "admin");
query = withPagination(query, 2, 20);
const result = await query;
```

Dynamic types per dialect: `PgSelect`, `MySqlSelect`, `SQLiteSelect`, `PgInsert`, `PgUpdate`, `PgDelete`, etc.

## Batch API

Execute multiple queries in a single round-trip (supported by Neon, Turso, Cloudflare D1, and others):

```ts
const db = drizzle(neon(url));

const results = await db.batch([
  db.insert(users).values({ name: "Alice" }),
  db.select().from(users),
  db.update(users).set({ name: "Bob" }).where(eq(users.id, 1)),
]);
// results[0] = insert result, results[1] = select result, etc.
```

## Read Replicas

Route reads to replicas and writes to the primary:

```ts
import { withReplicas } from "drizzle-orm/pg-core";

const primaryDb = drizzle("postgres://primary...");
const read1 = drizzle("postgres://replica1...");
const read2 = drizzle("postgres://replica2...");

const db = withReplicas(primaryDb, [read1, read2]);

// Reads go to random replica
await db.select().from(users);

// Writes go to primary
await db.insert(users).values({ name: "Alice" });

// Force read from primary
await db.$primary.select().from(users);
```

## Custom Types

Define custom column types for special database types:

```ts
import { customType } from "drizzle-orm/pg-core";

const customJsonb = customType<{
  data: Record<string, unknown>;
  driverData: string;
}>({
  dataType() {
    return "jsonb";
  },
  toDriver(value) {
    return JSON.stringify(value);
  },
  fromDriver(value) {
    if (typeof value === "string") return JSON.parse(value);
    return value as Record<string, unknown>;
  },
});
```

## Row-Level Security (PostgreSQL)

```ts
import { pgTable, pgPolicy, pgRole, integer, text } from "drizzle-orm/pg-core";
import { sql } from "drizzle-orm";

const admin = pgRole("admin");

// v1+ (beta): use pgTable.withRLS for RLS without policies
export const users = pgTable.withRLS("users", {
  id: integer().primaryKey(),
});

// Table with policies
export const posts = pgTable("posts", {
  id: integer().primaryKey(),
  authorId: integer("author_id"),
  content: text(),
}, (t) => [
  pgPolicy("author_policy", {
    as: "permissive",       // "permissive" | "restrictive"
    to: admin,              // role or "public"
    for: "all",             // "all" | "select" | "insert" | "update" | "delete"
    using: sql`${t.authorId} = current_user_id()`,
    withCheck: sql`${t.authorId} = current_user_id()`,
  }),
]);
```

## Conditional Filters Pattern

```ts
async function getUsers(filters: { name?: string; age?: number; role?: string }) {
  const conditions = [];
  if (filters.name) conditions.push(like(users.name, `%${filters.name}%`));
  if (filters.age) conditions.push(gte(users.age, filters.age));
  if (filters.role) conditions.push(eq(users.role, filters.role));

  return db.select().from(users)
    .where(conditions.length ? and(...conditions) : undefined);
}
```

## Prepared Statements

```ts
const prepared = db.select().from(users)
  .where(eq(users.id, sql.placeholder("id")))
  .prepare("get_user");

const user = await prepared.execute({ id: 1 });
```

## Query Utils

```ts
import { getTableColumns } from "drizzle-orm";

// Get all columns of a table
const columns = getTableColumns(users);
// { id: PgColumn, name: PgColumn, ... }

// Useful for partial selects excluding specific columns
const { password, ...rest } = getTableColumns(users);
const result = await db.select(rest).from(users);
```

<!--
Source references:
- https://orm.drizzle.team/docs/dynamic-query-building
- https://orm.drizzle.team/docs/batch-api
- https://orm.drizzle.team/docs/read-replicas
- https://orm.drizzle.team/docs/custom-types
- https://orm.drizzle.team/docs/rls
- https://orm.drizzle.team/docs/guides/conditional-filters-in-query
- https://orm.drizzle.team/docs/query-utils
-->
