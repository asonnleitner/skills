---
name: guides-conditional-filters
description: Building dynamic WHERE clauses with conditional and composable filters
---

# Conditional Filters

## Inline Conditional

Pass `undefined` to skip a filter — `and()`/`or()` ignore `undefined` arguments:

```typescript
import { and, gt, ilike, inArray } from 'drizzle-orm';

const searchPosts = async (term?: string, categories: string[] = [], minViews = 0) => {
  return db.select().from(posts).where(
    and(
      term ? ilike(posts.title, `%${term}%`) : undefined,
      categories.length > 0 ? inArray(posts.category, categories) : undefined,
      minViews > 0 ? gt(posts.views, minViews) : undefined,
    ),
  );
};

await searchPosts('AI', ['Tech'], 100);
await searchPosts(); // No filters, returns all
```

## Building Filter Arrays

```typescript
import { SQL, and, ilike, inArray, gt } from 'drizzle-orm';

const filters: SQL[] = [];

if (term) filters.push(ilike(posts.title, `%${term}%`));
if (categories.length) filters.push(inArray(posts.category, categories));
if (minViews) filters.push(gt(posts.views, minViews));

await db.select().from(posts).where(and(...filters));
```

## Custom Filter Operators

```typescript
import { AnyColumn, sql } from 'drizzle-orm';

const lenlt = (column: AnyColumn, value: number) => {
  return sql`length(${column}) < ${value}`;
};

await db.select().from(posts).where(
  and(
    maxLen ? lenlt(posts.title, maxLen) : undefined,
    views > 100 ? gt(posts.views, views) : undefined,
  ),
);
```

## Update Many with Different Values

Use CASE statements to update multiple rows with different values:

```typescript
import { SQL, inArray, sql } from 'drizzle-orm';

const inputs = [
  { id: 1, city: 'New York' },
  { id: 2, city: 'Los Angeles' },
  { id: 3, city: 'Chicago' },
];

const sqlChunks: SQL[] = [];
const ids: number[] = [];

sqlChunks.push(sql`(case`);
for (const input of inputs) {
  sqlChunks.push(sql`when ${users.id} = ${input.id} then ${input.city}`);
  ids.push(input.id);
}
sqlChunks.push(sql`end)`);

const finalSql = sql.join(sqlChunks, sql.raw(' '));
await db.update(users).set({ city: finalSql }).where(inArray(users.id, ids));
```

<!--
Source references:
- https://orm.drizzle.team/docs/guides/conditional-filters-in-query
- https://orm.drizzle.team/docs/guides/update-many-with-different-value
-->
