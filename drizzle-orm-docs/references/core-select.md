---
name: core-select
description: SELECT queries in Drizzle ORM including filters, ordering, pagination, aggregations, and CTEs
---

# Select Queries

## Basic Select

```typescript
// All columns
const result = await db.select().from(users);

// Partial select
const result = await db.select({
  id: users.id,
  name: users.name,
}).from(users);

// With SQL expressions
const result = await db.select({
  id: users.id,
  lowerName: sql<string>`lower(${users.name})`,
}).from(users);
```

## Distinct

```typescript
await db.selectDistinct().from(users).orderBy(users.id);

// PostgreSQL only: DISTINCT ON
await db.selectDistinctOn([users.id]).from(users).orderBy(users.id);
```

## getColumns Helper

```typescript
import { getColumns, sql } from 'drizzle-orm';

// All columns + extra computed column
await db.select({
  ...getColumns(posts),
  titleLength: sql<number>`length(${posts.title})`,
}).from(posts);

// Exclude specific columns
const { content, ...rest } = getColumns(posts);
await db.select({ ...rest }).from(posts);
```

## Filtering

```typescript
import { eq, ne, gt, gte, lt, lte, and, or, not,
  isNull, isNotNull, inArray, notInArray,
  between, notBetween, like, ilike } from 'drizzle-orm';

await db.select().from(users).where(eq(users.id, 42));
await db.select().from(users).where(and(gt(users.age, 18), lt(users.age, 65)));
await db.select().from(users).where(or(eq(users.role, 'admin'), eq(users.role, 'mod')));
await db.select().from(users).where(inArray(users.id, [1, 2, 3]));
await db.select().from(users).where(like(users.name, '%john%'));
await db.select().from(users).where(between(users.age, 18, 65));
```

Conditional filtering (pass `undefined` to skip a filter):

```typescript
const searchPosts = async (term?: string) => {
  await db.select().from(posts)
    .where(term ? ilike(posts.title, term) : undefined);
};
```

## Limit, Offset & Order By

```typescript
import { asc, desc } from 'drizzle-orm';

await db.select().from(users).orderBy(asc(users.name)).limit(10).offset(20);
await db.select().from(users).orderBy(desc(users.createdAt), asc(users.id));
```

## Aggregations

```typescript
import { count, countDistinct, avg, sum, min, max } from 'drizzle-orm';

await db.select({ total: count() }).from(users);
await db.select({ value: avg(users.age) }).from(users);
await db.select({ value: sum(orders.amount) }).from(orders);

// Group By + Having
await db.select({
  age: users.age,
  count: sql<number>`cast(count(${users.id}) as int)`,
}).from(users)
  .groupBy(users.age)
  .having(({ count }) => gt(count, 1));
```

## CTEs (WITH Clause)

```typescript
const sq = db.$with('sq').as(db.select().from(users).where(eq(users.id, 42)));
const result = await db.with(sq).select().from(sq);
```

## Subqueries

```typescript
const sq = db.select().from(users).where(eq(users.id, 42)).as('sq');
const result = await db.select().from(sq);

// In joins
const result = await db.select().from(users).leftJoin(sq, eq(users.id, sq.id));
```

## Conditional Select

```typescript
async function selectUsers(withName: boolean) {
  return db.select({
    id: users.id,
    ...(withName ? { name: users.name } : {}),
  }).from(users);
}
```

<!--
Source references:
- https://orm.drizzle.team/docs/select
-->
