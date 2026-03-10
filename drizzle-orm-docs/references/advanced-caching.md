---
name: advanced-caching
description: Query caching with Upstash Redis and custom cache implementations
---

# Caching

Drizzle supports query-level caching with Upstash Redis or custom implementations.

## Upstash Redis Cache

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import { upstashCache } from 'drizzle-orm/cache/upstash';

const db = drizzle({
  connection: process.env.DATABASE_URL,
  cache: upstashCache({
    url: process.env.UPSTASH_REDIS_URL,
    token: process.env.UPSTASH_REDIS_TOKEN,
  }),
});
```

## Query-Level Cache Control

```typescript
// Enable cache for a specific query
const users = await db.select().from(usersTable).$withCache();

// With custom TTL (seconds)
const users = await db.select().from(usersTable).$withCache({ ttl: 3600 });

// Custom cache key
const users = await db.select().from(usersTable).$withCache({ tag: 'all-users' });

// Disable cache for a query (when global cache is enabled)
const users = await db.select().from(usersTable).$withCache(false);
```

## Global vs Explicit Caching

```typescript
// Global: all queries cached by default
const db = drizzle({
  connection: process.env.DATABASE_URL,
  cache: upstashCache({ /* ... */ }),
  cacheConfig: { global: true, ttl: 600 },
});

// Explicit: only queries with $withCache() are cached (default)
const db = drizzle({
  connection: process.env.DATABASE_URL,
  cache: upstashCache({ /* ... */ }),
});
```

## Custom Cache Implementation

```typescript
const db = drizzle({
  connection: process.env.DATABASE_URL,
  cache: {
    async get(key: string) {
      // Return cached value or undefined
      return myCache.get(key);
    },
    async put(key: string, value: any, ttl?: number) {
      myCache.set(key, value, ttl);
    },
    async invalidate(tags: string[]) {
      for (const tag of tags) myCache.delete(tag);
    },
  },
});
```

<!--
Source references:
- https://orm.drizzle.team/docs/cache
-->
