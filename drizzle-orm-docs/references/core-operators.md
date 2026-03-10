---
name: core-operators
description: Filter and logical operators for WHERE clauses in Drizzle ORM
---

# Operators

All operators are imported from `drizzle-orm`.

## Comparison Operators

```typescript
import { eq, ne, gt, gte, lt, lte } from "drizzle-orm";

db.select().from(table).where(eq(table.column, 5));      // column = 5
db.select().from(table).where(ne(table.column, 5));      // column <> 5
db.select().from(table).where(gt(table.column, 5));      // column > 5
db.select().from(table).where(gte(table.column, 5));     // column >= 5
db.select().from(table).where(lt(table.column, 5));      // column < 5
db.select().from(table).where(lte(table.column, 5));     // column <= 5

// Column-to-column comparison
db.select().from(table).where(eq(table.col1, table.col2));
```

## Null Checks

```typescript
import { isNull, isNotNull } from "drizzle-orm";

db.select().from(table).where(isNull(table.column));
db.select().from(table).where(isNotNull(table.column));
```

## Array & Range Operators

```typescript
import { inArray, notInArray, between, notBetween } from "drizzle-orm";

db.select().from(table).where(inArray(table.column, [1, 2, 3]));
db.select().from(table).where(notInArray(table.column, [1, 2, 3]));
db.select().from(table).where(between(table.column, 2, 7));
db.select().from(table).where(notBetween(table.column, 2, 7));

// inArray with subquery
const query = db.select({ data: table2.column }).from(table2);
db.select().from(table).where(inArray(table.column, query));
```

## Pattern Matching

```typescript
import { like, ilike, notLike, notIlike } from "drizzle-orm";

db.select().from(table).where(like(table.column, "%llo wor%"));
db.select().from(table).where(ilike(table.column, "%llo wor%")); // PostgreSQL only
```

## Logical Operators

```typescript
import { and, or, not } from "drizzle-orm";

db.select().from(table).where(and(gt(table.col, 5), lt(table.col, 10)));
db.select().from(table).where(or(eq(table.col, 'a'), eq(table.col, 'b')));
db.select().from(table).where(not(eq(table.col, 5)));
```

`and()` and `or()` accept `undefined` values, which are filtered out — useful for conditional filtering:

```typescript
db.select().from(users).where(
  and(
    term ? ilike(users.name, term) : undefined,
    role ? eq(users.role, role) : undefined,
  )
);
```

## Exists

```typescript
import { exists, notExists } from "drizzle-orm";

const subquery = db.select().from(posts).where(eq(posts.userId, users.id));
db.select().from(users).where(exists(subquery));
db.select().from(users).where(notExists(subquery));
```

## PostgreSQL Array Operators

```typescript
import { arrayContains, arrayContained, arrayOverlaps } from "drizzle-orm";

// tags @> ['Typescript', 'ORM']
db.select().from(posts).where(arrayContains(posts.tags, ['Typescript', 'ORM']));

// tags <@ ['Typescript', 'ORM']
db.select().from(posts).where(arrayContained(posts.tags, ['Typescript', 'ORM']));

// tags && ['Typescript', 'ORM']
db.select().from(posts).where(arrayOverlaps(posts.tags, ['Typescript', 'ORM']));
```

## Set Operations

```typescript
import { union, unionAll, intersect, intersectAll, except, exceptAll } from 'drizzle-orm/pg-core';

// UNION
await db.select({ name: users.name }).from(users)
  .union(db.select({ name: customers.name }).from(customers))
  .limit(10);

// Function-style (also available: unionAll, intersect, intersectAll, except, exceptAll)
await union(
  db.select({ name: users.name }).from(users),
  db.select({ name: customers.name }).from(customers),
).limit(10);
```

<!--
Source references:
- https://orm.drizzle.team/docs/operators
- https://orm.drizzle.team/docs/set-operations
-->
