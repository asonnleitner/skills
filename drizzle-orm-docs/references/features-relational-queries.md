---
name: features-relational-queries
description: Relational Query Builder for fetching nested data with relations, filters, and pagination
---

# Relational Query Builder

Query related data using the `db.query` API. Requires passing schema to `drizzle()`:

```typescript
import * as schema from './schema';
const db = drizzle({ connection: process.env.DATABASE_URL, schema });
```

## findMany & findFirst

```typescript
const allUsers = await db.query.users.findMany();

const firstUser = await db.query.users.findFirst();
// Adds LIMIT 1 automatically
```

## Including Relations

```typescript
const usersWithPosts = await db.query.users.findMany({
  with: {
    posts: true,
  },
});

// Nested relations
const usersWithPostsAndComments = await db.query.users.findMany({
  with: {
    posts: {
      with: {
        comments: true,
      },
    },
  },
});
```

## Partial Column Selection

```typescript
// Include specific columns
await db.query.users.findMany({
  columns: {
    id: true,
    name: true,
  },
});

// Exclude columns
await db.query.users.findMany({
  columns: {
    password: false,
  },
});

// Nested column selection
await db.query.posts.findMany({
  columns: { id: true },
  with: {
    comments: {
      columns: { text: true },
    },
  },
});
```

## Filtering

```typescript
await db.query.users.findMany({
  where: (users, { eq }) => eq(users.id, 1),
});

// Multiple conditions
await db.query.users.findMany({
  where: (users, { and, gt, eq }) => and(gt(users.age, 18), eq(users.active, true)),
});
```

## Ordering

```typescript
await db.query.users.findMany({
  orderBy: (users, { asc }) => asc(users.createdAt),
});

// Multiple order
await db.query.users.findMany({
  orderBy: (users, { asc, desc }) => [desc(users.createdAt), asc(users.id)],
});
```

## Limit & Offset

```typescript
await db.query.users.findMany({
  limit: 10,
  offset: 20,
});

// Also works inside nested relations
await db.query.users.findMany({
  with: {
    posts: {
      limit: 5,
      orderBy: (posts, { desc }) => desc(posts.createdAt),
    },
  },
});
```

## Extras (Computed Fields)

```typescript
import { sql } from 'drizzle-orm';

await db.query.users.findMany({
  extras: {
    fullName: sql<string>`${users.firstName} || ' ' || ${users.lastName}`.as('full_name'),
  },
});

// Callback form
await db.query.users.findMany({
  extras: (table, { sql }) => ({
    lowerName: sql<string>`lower(${table.name})`.as('lower_name'),
  }),
});
```

## Prepared Statements

```typescript
const prepared = db.query.users.findMany({
  where: (users, { eq }) => eq(users.id, sql.placeholder('id')),
  limit: sql.placeholder('limit'),
}).prepare('get_users');

await prepared.execute({ id: 1, limit: 10 });
```

## Relational Queries v2 (Beta)

The v2 API uses object syntax instead of callbacks:

```typescript
await db.query.users.findMany({
  where: { id: 1 },                    // shorthand for eq
  where: { id: { gt: 5 } },            // comparison operators
  where: { OR: [{ id: 1 }, { id: 2 }] }, // logical OR
  orderBy: { createdAt: 'desc' },       // string direction
  limit: 10,
});
```

Available v2 filter operators: `eq`, `ne`, `gt`, `gte`, `lt`, `lte`, `in`, `notIn`, `like`, `ilike`, `notLike`, `notIlike`, `isNull`, `isNotNull`, `between`, `notBetween`, `arrayOverlaps`, `arrayContains`, `arrayContained`, `NOT`, `OR`, `AND`, `RAW`.

<!--
Source references:
- https://orm.drizzle.team/docs/rqb
- https://orm.drizzle.team/docs/rqb-v2
-->
