---
name: guides-pagination
description: Limit/offset and cursor-based pagination patterns in Drizzle ORM
---

# Pagination

## Limit/Offset Pagination

Simple but can be slow on large datasets:

```typescript
import { asc } from 'drizzle-orm';

const getUsers = async (page = 1, pageSize = 10) => {
  return db.select().from(users)
    .orderBy(asc(users.id))
    .limit(pageSize)
    .offset((page - 1) * pageSize);
};
```

### Deferred Join Technique

For large tables, join on a subquery that fetches only IDs:

```typescript
const getUsers = async (page = 1, pageSize = 10) => {
  const sq = db.select({ id: users.id }).from(users)
    .orderBy(users.id)
    .limit(pageSize)
    .offset((page - 1) * pageSize)
    .as('subquery');

  return db.select().from(users).innerJoin(sq, eq(users.id, sq.id)).orderBy(users.id);
};
```

## Cursor-Based Pagination

More efficient for large datasets, provides consistent results:

```typescript
import { asc, gt } from 'drizzle-orm';

const nextPage = async (cursor?: number, pageSize = 10) => {
  return db.select().from(users)
    .where(cursor ? gt(users.id, cursor) : undefined)
    .limit(pageSize)
    .orderBy(asc(users.id));
};

// First page
const page1 = await nextPage();
// Next page using last item's id as cursor
const page2 = await nextPage(page1[page1.length - 1].id);
```

### Multi-Column Cursor

For non-unique sort columns, use a composite cursor:

```typescript
import { and, asc, eq, gt, or } from 'drizzle-orm';

const nextPage = async (cursor?: { id: number; firstName: string }, pageSize = 10) => {
  return db.select().from(users)
    .where(cursor
      ? or(
          gt(users.firstName, cursor.firstName),
          and(eq(users.firstName, cursor.firstName), gt(users.id, cursor.id)),
        )
      : undefined
    )
    .limit(pageSize)
    .orderBy(asc(users.firstName), asc(users.id));
};
```

Create indexes for cursor columns:

```typescript
export const users = pgTable('users', { /* ... */ },
  (t) => [
    index('first_name_index').on(t.firstName).asc(),
    index('first_name_and_id_index').on(t.firstName, t.id).asc(),
  ]
);
```

### With Relational Queries

```typescript
const nextPage = async (cursor?: number, pageSize = 10) => {
  return db.query.users.findMany({
    where: (users, { gt }) => cursor ? gt(users.id, cursor) : undefined,
    orderBy: (users, { asc }) => asc(users.id),
    limit: pageSize,
  });
};
```

## Reusable Pagination Helper

```typescript
import { PgSelect } from 'drizzle-orm/pg-core';

function withPagination<T extends PgSelect>(
  qb: T,
  orderByColumn: PgColumn | SQL,
  page = 1,
  pageSize = 10,
) {
  return qb.orderBy(orderByColumn).limit(pageSize).offset((page - 1) * pageSize);
}

const query = db.select().from(users).$dynamic();
await withPagination(query, asc(users.id), 2, 10);
```

<!--
Source references:
- https://orm.drizzle.team/docs/guides/cursor-based-pagination
- https://orm.drizzle.team/docs/guides/limit-offset-pagination
-->
