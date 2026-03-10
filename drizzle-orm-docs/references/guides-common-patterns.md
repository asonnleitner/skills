---
name: guides-common-patterns
description: Common query patterns including counting, column selection, timestamps, and unique emails
---

# Common Patterns

## Counting Rows

```typescript
import { count } from 'drizzle-orm';

// Count all rows
await db.select({ count: count() }).from(products);

// Count with condition
await db.select({ count: count() }).from(products).where(gt(products.price, 100));

// Count with group by
await db.select({
  country: countries.name,
  citiesCount: count(cities.id),
}).from(countries)
  .leftJoin(cities, eq(countries.id, cities.countryId))
  .groupBy(countries.id);
```

For PostgreSQL/MySQL where count returns bigint, cast to int:

```typescript
const customCount = (column?: AnyColumn) =>
  column
    ? sql<number>`cast(count(${column}) as integer)`
    : sql<number>`cast(count(*) as integer)`;
```

## Include / Exclude Columns

```typescript
import { getColumns } from 'drizzle-orm';

// All columns + extra
await db.select({
  ...getColumns(posts),
  titleLength: sql<number>`length(${posts.title})`,
}).from(posts);

// Exclude columns
const { content, ...rest } = getColumns(posts);
await db.select({ ...rest }).from(posts);

// Relational queries — exclude
await db.query.posts.findMany({ columns: { content: false } });

// Relational queries — include specific
await db.query.posts.findMany({ columns: { title: true, id: true } });
```

## Timestamp Defaults

**PostgreSQL:**
```typescript
createdAt: timestamp('created_at').notNull().defaultNow(),
// Or unix epoch:
createdAt: integer('created_at').notNull().default(sql`extract(epoch from now())`),
```

**MySQL:**
```typescript
createdAt: timestamp('created_at').notNull().defaultNow(),
// With fractional seconds:
createdAt: timestamp('created_at', { fsp: 3 }).notNull().default(sql`now(3)`),
```

**SQLite:**
```typescript
createdAt: text('created_at').notNull().default(sql`(current_timestamp)`),
// Unix epoch:
createdAt: integer('created_at', { mode: 'timestamp' }).notNull().default(sql`(unixepoch())`),
```

## Empty Array Defaults

**PostgreSQL:**
```typescript
tags: text('tags').array().notNull().default(sql`'{}'::text[]`),
```

**MySQL (JSON):**
```typescript
tags: json('tags').$type<string[]>().notNull().default([]),
```

**SQLite (JSON):**
```typescript
tags: text('tags', { mode: 'json' }).$type<string[]>().notNull().default(sql`(json_array())`),
```

## Case-Insensitive Unique Email

```typescript
import { SQL, sql } from 'drizzle-orm';
import { AnyPgColumn, uniqueIndex } from 'drizzle-orm/pg-core';

export function lower(col: AnyPgColumn): SQL {
  return sql`lower(${col})`;
}

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull(),
}, (table) => [
  uniqueIndex('email_unique_idx').on(lower(table.email)),
]);

// Query
await db.select().from(users).where(eq(lower(users.email), email.toLowerCase()));
```

## Select Parents with Related Children

```typescript
import { exists } from 'drizzle-orm';

// Only users who have at least one post
const sq = db.select({ id: sql`1` }).from(posts).where(eq(posts.userId, users.id));
await db.select().from(users).where(exists(sq));
```

<!--
Source references:
- https://orm.drizzle.team/docs/guides/count-rows
- https://orm.drizzle.team/docs/guides/include-or-exclude-columns
- https://orm.drizzle.team/docs/guides/timestamp-default-value
- https://orm.drizzle.team/docs/guides/empty-array-default-value
- https://orm.drizzle.team/docs/guides/unique-case-insensitive-email
- https://orm.drizzle.team/docs/guides/select-parent-rows-with-at-least-one-related-child-row
-->
