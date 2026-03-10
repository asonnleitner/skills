---
name: advanced-performance
description: Performance optimization with prepared statements, serverless tips, and read replicas
---

# Performance

## Prepared Statements

Precompile queries for reuse across multiple executions:

```typescript
const prepared = db.select().from(users)
  .where(eq(users.id, sql.placeholder('id')))
  .prepare('get_user');

// Execute multiple times with different params
await prepared.execute({ id: 1 });
await prepared.execute({ id: 42 });
```

With relational queries:

```typescript
const prepared = db.query.users.findMany({
  where: (users, { eq }) => eq(users.id, sql.placeholder('id')),
  limit: sql.placeholder('limit'),
}).prepare('find_users');

await prepared.execute({ id: 1, limit: 10 });
```

## Serverless Optimization

In serverless environments (Lambda, Vercel, Cloudflare Workers), connections and prepared statements persist within a single function invocation (~15 minutes on Lambda).

Key tips:
- Initialize `drizzle()` outside the handler to reuse across warm invocations
- Use prepared statements for frequently executed queries
- Consider HTTP-based drivers (Neon HTTP, D1) for edge deployments

## Read Replicas

Split reads and writes across primary and replica databases:

```typescript
import { withReplicas } from 'drizzle-orm';

const primary = drizzle(primaryUrl);
const replica1 = drizzle(replicaUrl1);
const replica2 = drizzle(replicaUrl2);

const db = withReplicas(primary, [replica1, replica2]);

// Reads go to random replica
await db.select().from(users);

// Writes go to primary
await db.insert(users).values({ name: 'John' });

// Force read from primary
await db.$primary.select().from(users);
```

Custom replica selection:

```typescript
const db = withReplicas(primary, [replica1, replica2], (replicas) => {
  // Custom selection logic (e.g., round-robin, weighted)
  return replicas[Math.floor(Math.random() * replicas.length)];
});
```

## Batch API

Execute multiple statements in a single round-trip (LibSQL, Neon HTTP, D1):

```typescript
// Neon HTTP
import { drizzle } from 'drizzle-orm/neon-http';

const db = drizzle(process.env.DATABASE_URL);

const results = await db.batch([
  db.insert(users).values({ name: 'John' }),
  db.select().from(users),
  db.update(users).set({ name: 'Jane' }).where(eq(users.id, 1)),
]);
```

<!--
Source references:
- https://orm.drizzle.team/docs/perf-queries
- https://orm.drizzle.team/docs/perf-serverless
- https://orm.drizzle.team/docs/read-replicas
- https://orm.drizzle.team/docs/batch-api
-->
