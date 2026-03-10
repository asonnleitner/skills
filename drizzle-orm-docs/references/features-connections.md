---
name: features-connections
description: Database connection patterns for different drivers in Drizzle ORM
---

# Database Connections

The `drizzle()` function accepts different argument patterns depending on the driver.

## Pattern 1: Connection String

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
const db = drizzle(process.env.DATABASE_URL);
```

Works with: `node-postgres`, `postgres-js`, `neon-http`, `neon-serverless`.

## Pattern 2: Existing Client

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const db = drizzle({ client: pool });
```

## Pattern 3: Connection Config

```typescript
import { drizzle } from 'drizzle-orm/libsql';
const db = drizzle({
  connection: {
    url: process.env.DATABASE_URL,
    authToken: process.env.DATABASE_AUTH_TOKEN,
  },
});
```

## Pattern 4: Proxy (Custom Driver)

```typescript
import { drizzle } from 'drizzle-orm/pg-proxy';

const db = drizzle(async (sql, params, method) => {
  const rows = await fetch('/query', {
    method: 'POST',
    body: JSON.stringify({ sql, params, method }),
  }).then(r => r.json());
  return { rows };
});
```

## Common Drivers

**PostgreSQL:** `drizzle-orm/node-postgres`, `drizzle-orm/postgres-js`, `drizzle-orm/neon-http`, `drizzle-orm/neon-serverless`, `drizzle-orm/pglite`

**MySQL:** `drizzle-orm/mysql2`

**SQLite:** `drizzle-orm/better-sqlite3`, `drizzle-orm/bun-sqlite`, `drizzle-orm/libsql`, `drizzle-orm/d1`

**Special cases:**
```typescript
// Supabase with connection pooling (transaction mode)
import postgres from 'postgres';
const client = postgres(process.env.DATABASE_URL, { prepare: false });
const db = drizzle({ client });

// Cloudflare D1
const db = drizzle({ connection: env.DB });

// PGlite (in-memory)
import { drizzle } from 'drizzle-orm/pglite';
const db = drizzle(); // in-memory

// Vercel Postgres
import { drizzle } from 'drizzle-orm/vercel-postgres';
const db = drizzle();
```

## Passing Schema

Required for relational queries:

```typescript
import * as schema from './schema';
const db = drizzle({ connection: process.env.DATABASE_URL, schema });
```

## Accessing Underlying Client

```typescript
const db = drizzle(process.env.DATABASE_URL);
const client = db.$client;
```

<!--
Source references:
- https://orm.drizzle.team/docs/connect-overview
-->
