---
name: features-views
description: Database views and materialized views in Drizzle ORM
---

# Views

## Declaring Views

```typescript
import { pgTable, pgView, serial, text, timestamp } from "drizzle-orm/pg-core";
import { eq } from "drizzle-orm";

export const user = pgTable("user", {
  id: serial(),
  name: text(),
  email: text(),
  role: text().$type<"admin" | "customer">(),
});

// View with inline query builder
export const customerView = pgView("customers_view").as((qb) =>
  qb.select().from(user).where(eq(user.role, "customer"))
);
```

Database-specific view functions:
- PostgreSQL: `pgView`, `pgMaterializedView`
- MySQL: `mysqlView`
- SQLite: `sqliteView`
- CockroachDB: `cockroachView`, `cockroachMaterializedView`

## Views with Raw SQL

Define schema shape manually when using raw SQL:

```typescript
const newYorkers = pgView('new_yorkers', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  cityId: integer('city_id').notNull(),
}).as(sql`select * from ${users} where ${eq(users.cityId, 1)}`);
```

## Existing Views

Reference views already in the database without managing their creation:

```typescript
export const trimmedUser = pgView("trimmed_user", {
  id: serial("id"),
  name: text("name"),
  email: text("email"),
}).existing();
```

## Materialized Views (PostgreSQL, CockroachDB)

```typescript
const newYorkers = pgMaterializedView('new_yorkers')
  .as((qb) => qb.select().from(users).where(eq(users.cityId, 1)));

// Refresh
await db.refreshMaterializedView(newYorkers);
await db.refreshMaterializedView(newYorkers).concurrently();
await db.refreshMaterializedView(newYorkers).withNoData();
```

## View Options (PostgreSQL)

```typescript
const view = pgView('my_view')
  .with({
    checkOption: 'cascaded',
    securityBarrier: true,
    securityInvoker: true,
  })
  .as((qb) => qb.select().from(users));
```

<!--
Source references:
- https://orm.drizzle.team/docs/views
-->
