---
name: core-delete
description: DELETE operations in Drizzle ORM including returning and CTEs
---

# Delete

## Basic Delete

```typescript
// Delete all rows
await db.delete(users);

// Delete with filter
await db.delete(users).where(eq(users.name, 'Dan'));
```

## Returning (PostgreSQL, SQLite, CockroachDB)

```typescript
const deletedUser = await db.delete(users)
  .where(eq(users.name, 'Dan'))
  .returning();

// Partial return
const deletedUserIds = await db.delete(users)
  .where(eq(users.name, 'Dan'))
  .returning({ deletedId: users.id });
```

## Limit & Order By (MySQL, SQLite)

```typescript
await db.delete(users).where(eq(users.name, 'Dan')).limit(2);
await db.delete(users).where(eq(users.name, 'Dan')).orderBy(desc(users.name));
```

## WITH Delete

```typescript
const averageAmount = db.$with('average_amount').as(
  db.select({ value: sql`avg(${orders.amount})`.as('value') }).from(orders)
);

const result = await db.with(averageAmount)
  .delete(orders)
  .where(gt(orders.amount, sql`(select * from ${averageAmount})`))
  .returning({ id: orders.id });
```

<!--
Source references:
- https://orm.drizzle.team/docs/delete
-->
