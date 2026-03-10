---
name: core-update
description: UPDATE operations in Drizzle ORM including returning, UPDATE...FROM, and CTEs
---

# Update

## Basic Update

```typescript
await db.update(users)
  .set({ name: 'Mr. Dan' })
  .where(eq(users.name, 'Dan'));

// With SQL expression
await db.update(users)
  .set({ updatedAt: sql`NOW()` })
  .where(eq(users.name, 'Dan'));
```

## Returning (PostgreSQL, SQLite, CockroachDB)

```typescript
const updatedUserId = await db.update(users)
  .set({ name: 'Mr. Dan' })
  .where(eq(users.name, 'Dan'))
  .returning({ updatedId: users.id });
```

## Limit & Order By (MySQL, SQLite)

```typescript
await db.update(users).set({ verified: true }).limit(2);
await db.update(users).set({ verified: true }).orderBy(desc(users.name));
```

## UPDATE ... FROM (PostgreSQL, SQLite, CockroachDB)

Join another table in an update:

```typescript
await db.update(users)
  .set({ cityId: cities.id })
  .from(cities)
  .where(and(eq(cities.name, 'Seattle'), eq(users.name, 'John')));

// With alias
const c = alias(cities, 'c');
await db.update(users).set({ cityId: c.id }).from(c);

// With returning
const updatedUsers = await db.update(users)
  .set({ cityId: cities.id })
  .from(cities)
  .returning({ id: users.id, cityName: cities.name });
```

## WITH Update

```typescript
const averagePrice = db.$with('average_price').as(
  db.select({ value: sql`avg(${products.price})`.as('value') }).from(products)
);

const result = await db.with(averagePrice)
  .update(products)
  .set({ cheap: true })
  .where(lt(products.price, sql`(select * from ${averagePrice})`))
  .returning({ id: products.id });
```

## Increment / Decrement Helpers

```typescript
import { AnyColumn, sql } from 'drizzle-orm';

const increment = (column: AnyColumn, value = 1) => sql`${column} + ${value}`;
const decrement = (column: AnyColumn, value = 1) => sql`${column} - ${value}`;

await db.update(table).set({
  counter1: increment(table.counter1),
  counter2: decrement(table.counter2, 10),
}).where(eq(table.id, 1));
```

## Toggle Boolean

```typescript
import { not } from 'drizzle-orm';

await db.update(table).set({ isActive: not(table.isActive) }).where(eq(table.id, 1));
```

<!--
Source references:
- https://orm.drizzle.team/docs/update
- https://orm.drizzle.team/docs/guides/incrementing-a-value
- https://orm.drizzle.team/docs/guides/decrementing-a-value
- https://orm.drizzle.team/docs/guides/toggling-a-boolean-field
-->
