---
name: guides-full-text-search
description: PostgreSQL full-text search and pgvector similarity search with Drizzle ORM
---

# Full-Text & Vector Search

## PostgreSQL Full-Text Search

### Schema with GIN Index

```typescript
import { index, pgTable, serial, text } from 'drizzle-orm/pg-core';
import { sql } from 'drizzle-orm';

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  description: text('description').notNull(),
}, (table) => [
  index('title_search_index').using(
    'gin', sql`to_tsvector('english', ${table.title})`
  ),
]);
```

### Query Patterns

```typescript
// Basic search (stemming applied)
await db.select().from(posts)
  .where(sql`to_tsvector('english', ${posts.title}) @@ to_tsquery('english', ${term})`);

// Match any keyword (OR): 'Europe | Asia'
// Match all keywords (AND) with plainto_tsquery:
await db.select().from(posts)
  .where(sql`to_tsvector('english', ${posts.title}) @@ plainto_tsquery('english', ${term})`);

// Exact phrase match:
await db.select().from(posts)
  .where(sql`to_tsvector('english', ${posts.title}) @@ phraseto_tsquery('english', ${term})`);

// Web search syntax (supports quotes, OR, -negation):
await db.select().from(posts)
  .where(sql`to_tsvector('english', ${posts.title}) @@ websearch_to_tsquery('english', ${term})`);
```

### Multi-Column with Weighted Ranking

```typescript
const matchQuery = sql`(
  setweight(to_tsvector('english', ${posts.title}), 'A') ||
  setweight(to_tsvector('english', ${posts.description}), 'B')
), to_tsquery('english', ${search})`;

await db.select({
  ...getColumns(posts),
  rank: sql`ts_rank(${matchQuery})`,
}).from(posts)
  .where(sql`(
    setweight(to_tsvector('english', ${posts.title}), 'A') ||
    setweight(to_tsvector('english', ${posts.description}), 'B')
  ) @@ to_tsquery('english', ${search})`)
  .orderBy((t) => desc(t.rank));
```

## Vector Similarity Search (pgvector)

### Setup

Create extension via custom migration:
```sql
CREATE EXTENSION vector;
```

### Schema

```typescript
import { index, pgTable, serial, text, vector } from 'drizzle-orm/pg-core';

export const guides = pgTable('guides', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  description: text('description').notNull(),
  embedding: vector('embedding', { dimensions: 1536 }),
}, (table) => [
  index('embedding_index').using('hnsw', table.embedding.op('vector_cosine_ops')),
]);
```

### Similarity Search

```typescript
import { cosineDistance, desc, gt, sql } from 'drizzle-orm';

const findSimilar = async (queryEmbedding: number[]) => {
  const similarity = sql<number>`1 - (${cosineDistance(guides.embedding, queryEmbedding)})`;

  return db.select({ title: guides.title, url: guides.url, similarity })
    .from(guides)
    .where(gt(similarity, 0.5))
    .orderBy((t) => desc(t.similarity))
    .limit(4);
};
```

Distance functions available: `cosineDistance`, `l2Distance`, `l1Distance`, `innerProduct`.

<!--
Source references:
- https://orm.drizzle.team/docs/guides/postgresql-full-text-search
- https://orm.drizzle.team/docs/guides/vector-similarity-search
-->
