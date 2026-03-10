---
name: drizzle-kit-migrations
description: Generating, applying, and managing migrations with Drizzle Kit
---

# Migrations

## Generate Migrations

Generate SQL migration files from schema changes:

```bash
npx drizzle-kit generate
```

Creates timestamped SQL files in the `out` directory (default `./drizzle/`):
```
📂 drizzle
 ├ 📂 meta
 ├ 📜 0000_create_users.sql
 └ 📜 0001_add_posts.sql
```

**Options:**
```bash
npx drizzle-kit generate --name custom_migration_name
npx drizzle-kit generate --config ./custom-config.ts
```

## Apply Migrations

Apply generated SQL migrations to the database:

```bash
npx drizzle-kit migrate
```

## Push (No Migration Files)

Apply schema directly without generating files — ideal for prototyping:

```bash
npx drizzle-kit push
```

**Options:**
```bash
npx drizzle-kit push --verbose  # Show SQL that will be executed
npx drizzle-kit push --strict   # Ask for confirmation before executing
npx drizzle-kit push --force    # Auto-accept data-loss statements
```

## Pull (Introspect)

Generate TypeScript schema from an existing database:

```bash
npx drizzle-kit pull
```

Outputs schema files in the `out` directory.

## Custom Migrations

Generate an empty migration file for custom SQL:

```bash
npx drizzle-kit generate --custom
```

Then write your custom SQL in the generated file:

```sql
-- 0002_custom.sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE INDEX idx_embedding ON items USING hnsw (embedding vector_cosine_ops);
```

## Runtime Migrations

Apply migrations programmatically in your application:

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import { migrate } from 'drizzle-orm/node-postgres/migrator';

const db = drizzle(process.env.DATABASE_URL);

await migrate(db, { migrationsFolder: './drizzle' });
```

## Check & Up

```bash
# Verify migration consistency
npx drizzle-kit check

# Upgrade internal snapshots after drizzle-kit version updates
npx drizzle-kit up
```

<!--
Source references:
- https://orm.drizzle.team/docs/drizzle-kit-generate
- https://orm.drizzle.team/docs/drizzle-kit-migrate
- https://orm.drizzle.team/docs/drizzle-kit-push
- https://orm.drizzle.team/docs/drizzle-kit-pull
- https://orm.drizzle.team/docs/kit-custom-migrations
- https://orm.drizzle.team/docs/migrations
-->
