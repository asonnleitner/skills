---
name: best-practices-patterns
description: Common Drizzle ORM patterns - pagination, upserts, full-text search, and schema validation with Zod
---

# Common Patterns

## Pagination

### Offset-based

```ts
async function getUsers(page: number, pageSize = 20) {
  return db.select().from(users)
    .orderBy(users.id)
    .limit(pageSize)
    .offset((page - 1) * pageSize);
}
```

### Cursor-based (recommended for large datasets)

```ts
async function getUsers(cursor?: number, pageSize = 20) {
  return db.select().from(users)
    .where(cursor ? gt(users.id, cursor) : undefined)
    .orderBy(asc(users.id))
    .limit(pageSize);
}
// Next cursor = last item's id
```

## Upserts

### PostgreSQL / SQLite - ON CONFLICT

```ts
await db.insert(users)
  .values({ id: 1, name: "Alice", email: "alice@example.com" })
  .onConflictDoUpdate({
    target: users.email,
    set: { name: "Alice Updated" },
  });

// With setWhere (conditional update)
await db.insert(users)
  .values({ id: 1, name: "Alice" })
  .onConflictDoUpdate({
    target: users.id,
    set: { name: sql`excluded.name` },
    setWhere: sql`${users.name} <> excluded.name`,
  });
```

### MySQL - ON DUPLICATE KEY UPDATE

```ts
await db.insert(users)
  .values({ id: 1, name: "Alice" })
  .onDuplicateKeyUpdate({ set: { name: "Alice Updated" } });
```

## Increment / Decrement Values

```ts
// Increment
await db.update(products)
  .set({ stock: sql`${products.stock} + 1` })
  .where(eq(products.id, 1));

// Decrement
await db.update(products)
  .set({ stock: sql`${products.stock} - 1` })
  .where(eq(products.id, 1));

// Toggle boolean
await db.update(users)
  .set({ active: sql`NOT ${users.active}` })
  .where(eq(users.id, 1));
```

## Include/Exclude Columns

```ts
import { getTableColumns } from "drizzle-orm";

// Exclude specific columns
const { password, secret, ...safeColumns } = getTableColumns(users);
const result = await db.select(safeColumns).from(users);
```

## Zod Schema Validation

```ts
import { createInsertSchema, createSelectSchema, createUpdateSchema } from "drizzle-zod";

const insertUserSchema = createInsertSchema(users, {
  email: (schema) => schema.email(),
  name: (schema) => schema.min(2).max(100),
});

const selectUserSchema = createSelectSchema(users);

// Usage
const validated = insertUserSchema.parse(requestBody);
await db.insert(users).values(validated);
```

Available packages: `drizzle-zod`, `drizzle-valibot`, `drizzle-typebox`, `drizzle-arktype`.

## PostgreSQL Full-Text Search

```ts
import { sql } from "drizzle-orm";
import { index, tsvector } from "drizzle-orm/pg-core";

// With generated column
export const posts = pgTable("posts", {
  id: integer().primaryKey(),
  title: text().notNull(),
  body: text().notNull(),
  searchVector: tsvector("search_vector")
    .generatedAlwaysAs(sql`to_tsvector('english', ${posts.title} || ' ' || ${posts.body})`),
}, (t) => [
  index("search_idx").using("gin", t.searchVector),
]);

// Query
await db.select().from(posts)
  .where(sql`${posts.searchVector} @@ to_tsquery('english', ${query})`);
```

## Vector Similarity Search (pgvector)

```ts
import { index, pgTable, vector } from "drizzle-orm/pg-core";
import { sql } from "drizzle-orm";

export const items = pgTable("items", {
  id: integer().primaryKey(),
  embedding: vector("embedding", { dimensions: 1536 }),
}, (t) => [
  index("embedding_idx").using("hnsw", t.embedding.op("vector_cosine_ops")),
]);

// Search
const similar = await db.select()
  .from(items)
  .orderBy(sql`${items.embedding} <=> ${sql`${embedding}::vector`}`)
  .limit(10);
```

## Database Seeding

```ts
import { seed } from "drizzle-seed";

await seed(db, { users, posts }).refine((f) => ({
  users: {
    count: 100,
    columns: {
      name: f.fullName(),
      email: f.email(),
    },
  },
  posts: {
    count: 500,
    columns: {
      content: f.loremIpsum({ sentenceCount: 3 }),
    },
  },
}));

// Reset (truncate + seed)
await reset(db, { users, posts });
```

<!--
Source references:
- https://orm.drizzle.team/docs/guides/cursor-based-pagination
- https://orm.drizzle.team/docs/guides/limit-offset-pagination
- https://orm.drizzle.team/docs/guides/upsert
- https://orm.drizzle.team/docs/guides/postgresql-full-text-search
- https://orm.drizzle.team/docs/guides/vector-similarity-search
- https://orm.drizzle.team/docs/guides/include-or-exclude-columns
- https://orm.drizzle.team/docs/zod
- https://orm.drizzle.team/docs/seed-overview
-->
