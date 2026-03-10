---
name: features-dynamic-queries
description: Dynamic query building patterns with $dynamic() for reusable query modifiers
---

# Dynamic Query Building

Use `.$dynamic()` to build queries incrementally with reusable helper functions.

## Basic Pattern

```typescript
function withPagination<T extends PgSelect>(
  qb: T,
  page = 1,
  pageSize = 10,
) {
  return qb.limit(pageSize).offset((page - 1) * pageSize);
}

const query = db.select().from(users).$dynamic();
const result = await withPagination(query, 2, 10);
```

## Type Imports

Each dialect has its own types:

| Dialect | Select Type | Query Builder Type |
|---------|-------------|-------------------|
| PostgreSQL | `PgSelect` | `PgSelectQueryBuilder` |
| MySQL | `MySqlSelect` | `MySqlSelectQueryBuilder` |
| SQLite | `SQLiteSelect` | `SQLiteSelectQueryBuilder` |

## Adding Joins Dynamically

When a helper adds a join, the query type changes. Use generics:

```typescript
import { PgSelect } from 'drizzle-orm/pg-core';

function withFriends<T extends PgSelect>(qb: T) {
  return qb.leftJoin(friends, eq(friends.userId, users.id));
}

const query = db.select().from(users).$dynamic();
const result = await withFriends(query);
```

## Combining Multiple Helpers

```typescript
import { SQL, asc } from 'drizzle-orm';
import { PgColumn, PgSelect } from 'drizzle-orm/pg-core';

function withPagination<T extends PgSelect>(
  qb: T,
  orderByColumn: PgColumn | SQL | SQL.Aliased,
  page = 1,
  pageSize = 3,
) {
  return qb
    .orderBy(orderByColumn)
    .limit(pageSize)
    .offset((page - 1) * pageSize);
}

const query = db.select().from(users).$dynamic();
await withPagination(query, asc(users.id), 2, 5);
```

<!--
Source references:
- https://orm.drizzle.team/docs/dynamic-query-building
-->
