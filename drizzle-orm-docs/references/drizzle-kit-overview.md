---
name: drizzle-kit-overview
description: Drizzle Kit CLI commands for migrations, schema push, pull, and studio
---

# Drizzle Kit

Drizzle Kit is the CLI companion for schema migrations and database management.

## Commands

| Command | Description |
|---------|-------------|
| `drizzle-kit generate` | Generate SQL migration files from schema changes |
| `drizzle-kit migrate` | Apply generated migrations to the database |
| `drizzle-kit push` | Push schema directly to database (no migration files) |
| `drizzle-kit pull` | Introspect database and generate TypeScript schema |
| `drizzle-kit check` | Check for consistency of generated migrations |
| `drizzle-kit up` | Upgrade internal migration snapshots |
| `drizzle-kit studio` | Launch Drizzle Studio GUI |

## Configuration File

`drizzle.config.ts` at project root:

```typescript
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  dialect: 'postgresql', // postgresql | mysql | sqlite | turso | singlestore | gel
  schema: './src/db/schema.ts',
  out: './drizzle', // migration output directory (default: ./drizzle)
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

**Schema as directory:**
```typescript
export default defineConfig({
  schema: './src/db/schema',  // reads all .ts files
});
```

**Schema as glob pattern:**
```typescript
export default defineConfig({
  schema: './src/**/*.sql.ts',
});
```

## Filtering

```typescript
export default defineConfig({
  // Only manage specific tables
  tablesFilter: ['users', 'posts'],
  // Or exclude with negation
  tablesFilter: ['!_*'],  // exclude tables starting with _

  // PostgreSQL: filter schemas
  schemaFilter: ['public', 'my_schema'],

  // PostgreSQL: filter extensions
  extensionsFilters: ['postgis'],
});
```

## Driver-Specific Configs

**Turso:**
```typescript
export default defineConfig({
  dialect: 'turso',
  dbCredentials: {
    url: process.env.TURSO_DATABASE_URL!,
    authToken: process.env.TURSO_AUTH_TOKEN,
  },
});
```

**Cloudflare D1:**
```typescript
export default defineConfig({
  dialect: 'sqlite',
  driver: 'd1-http',
  dbCredentials: {
    accountId: process.env.CLOUDFLARE_ACCOUNT_ID!,
    databaseId: process.env.CLOUDFLARE_DATABASE_ID!,
    token: process.env.CLOUDFLARE_D1_TOKEN!,
  },
});
```

**AWS Data API:**
```typescript
export default defineConfig({
  dialect: 'postgresql',
  driver: 'aws-data-api',
  dbCredentials: {
    database: process.env.DATABASE!,
    resourceArn: process.env.RESOURCE_ARN!,
    secretArn: process.env.SECRET_ARN!,
  },
});
```

## Migration Approaches

1. **Code-first with push** — `drizzle-kit push` (prototyping, no migration files)
2. **Code-first with generate/migrate** — `drizzle-kit generate` + `drizzle-kit migrate` (production)
3. **Database-first** — `drizzle-kit pull` (existing database)

<!--
Source references:
- https://orm.drizzle.team/docs/kit-overview
- https://orm.drizzle.team/docs/drizzle-config-file
- https://orm.drizzle.team/docs/migrations
-->
