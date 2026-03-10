---
name: core-queries
description: Drizzle ORM SQL query builder - select, insert, update, delete with type-safe operators
---

# SQL Queries

Drizzle provides an SQL-like, type-safe query builder. All examples use PostgreSQL; MySQL and SQLite patterns are identical.

## Select

```ts
import { eq, gt, and, or, like, sql } from "drizzle-orm";

// Select all
const allUsers = await db.select().from(users);

// Partial select
const result = await db.select({
  id: users.id,
  name: users.name,
}).from(users);

// With expressions
const result = await db.select({
  id: users.id,
  lowerName: sql<string>`lower(${users.name})`,
}).from(users);

// Where clause
await db.select().from(users).where(eq(users.id, 1));
await db.select().from(users).where(and(gt(users.age, 18), eq(users.role, "admin")));
await db.select().from(users).where(or(eq(users.role, "admin"), eq(users.role, "moderator")));

// Distinct
await db.selectDistinct().from(users);
await db.selectDistinctOn([users.id]).from(users); // PG only

// Order, limit, offset
await db.select().from(users)
  .orderBy(users.name)           // ASC by default
  .orderBy(desc(users.createdAt))
  .limit(10)
  .offset(20);

// Count
const count = await db.$count(users);
const filtered = await db.$count(users, eq(users.role, "admin"));

// Group by + having
await db.select({
  role: users.role,
  count: sql<number>`count(*)`,
}).from(users)
  .groupBy(users.role)
  .having(gt(sql`count(*)`, 5));

// Subquery
const sq = db.select({ avgAge: sql`avg(${users.age})`.as("avg_age") }).from(users).as("sq");
await db.select().from(users).where(gt(users.age, sq.avgAge));
```

## Insert

```ts
// Single insert
await db.insert(users).values({ name: "Alice", email: "alice@example.com" });

// Multiple insert
await db.insert(users).values([
  { name: "Alice", email: "alice@example.com" },
  { name: "Bob", email: "bob@example.com" },
]);

// Returning (PG, SQLite)
const [inserted] = await db.insert(users).values({ name: "Alice" }).returning();
const [{ id }] = await db.insert(users).values({ name: "Alice" }).returning({ id: users.id });

// MySQL returning ID
const result = await db.insert(users).values({ name: "Alice" }).$returningId();

// Upsert - on conflict do nothing (PG, SQLite)
await db.insert(users)
  .values({ id: 1, name: "Alice" })
  .onConflictDoNothing({ target: users.id });

// Upsert - on conflict do update (PG, SQLite)
await db.insert(users)
  .values({ id: 1, name: "Alice" })
  .onConflictDoUpdate({
    target: users.id,
    set: { name: "Alice Updated" },
  });

// MySQL upsert - on duplicate key update
await db.insert(users)
  .values({ id: 1, name: "Alice" })
  .onDuplicateKeyUpdate({ set: { name: "Alice Updated" } });

// Insert from select
await db.insert(archivedUsers).select(
  db.select().from(users).where(eq(users.archived, true))
);
```

## Update

```ts
await db.update(users)
  .set({ name: "Updated Name" })
  .where(eq(users.id, 1));

// With SQL expressions
await db.update(accounts)
  .set({ balance: sql`${accounts.balance} + 100` })
  .where(eq(accounts.id, 1));

// Returning (PG, SQLite)
const [updated] = await db.update(users)
  .set({ name: "New Name" })
  .where(eq(users.id, 1))
  .returning();
```

## Delete

```ts
await db.delete(users).where(eq(users.id, 1));

// Returning (PG, SQLite)
const [deleted] = await db.delete(users)
  .where(eq(users.id, 1))
  .returning();

// Delete all rows
await db.delete(users);
```

## Filter Operators

All imported from `"drizzle-orm"`:

| Operator | SQL | Usage |
|----------|-----|-------|
| `eq(col, val)` | `=` | `eq(users.id, 1)` |
| `ne(col, val)` | `<>` | `ne(users.role, "admin")` |
| `gt(col, val)` | `>` | `gt(users.age, 18)` |
| `gte(col, val)` | `>=` | `gte(users.age, 18)` |
| `lt(col, val)` | `<` | `lt(users.age, 65)` |
| `lte(col, val)` | `<=` | `lte(users.age, 65)` |
| `isNull(col)` | `IS NULL` | `isNull(users.deletedAt)` |
| `isNotNull(col)` | `IS NOT NULL` | `isNotNull(users.email)` |
| `inArray(col, vals)` | `IN` | `inArray(users.id, [1, 2, 3])` |
| `notInArray(col, vals)` | `NOT IN` | `notInArray(users.id, [1, 2])` |
| `between(col, a, b)` | `BETWEEN` | `between(users.age, 18, 65)` |
| `like(col, pat)` | `LIKE` | `like(users.name, "%john%")` |
| `ilike(col, pat)` | `ILIKE` | `ilike(users.name, "%john%")` (PG) |
| `exists(subquery)` | `EXISTS` | `exists(db.select().from(...))` |
| `not(expr)` | `NOT` | `not(eq(users.id, 1))` |
| `and(...exprs)` | `AND` | `and(eq(...), gt(...))` |
| `or(...exprs)` | `OR` | `or(eq(...), eq(...))` |
| `arrayContains()` | `@>` | PG array operator |
| `arrayContained()` | `<@` | PG array operator |
| `arrayOverlaps()` | `&&` | PG array operator |

## The `sql` Template Tag

Use for raw SQL expressions:

```ts
import { sql } from "drizzle-orm";

// In select
db.select({ total: sql<number>`count(*)` }).from(users);

// In where
db.select().from(users).where(sql`${users.age} > 18`);

// Referencing columns safely
sql`lower(${users.name})`;

// With mapWith for runtime transformation
sql`count(*)`.mapWith(Number);

// sql.raw() for non-parameterized SQL (careful with injection!)
sql.raw(`DROP TABLE IF EXISTS users`);
```

<!--
Source references:
- https://orm.drizzle.team/docs/select
- https://orm.drizzle.team/docs/insert
- https://orm.drizzle.team/docs/update
- https://orm.drizzle.team/docs/delete
- https://orm.drizzle.team/docs/operators
- https://orm.drizzle.team/docs/sql
-->
