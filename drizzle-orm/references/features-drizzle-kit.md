---
name: features-drizzle-kit
description: Drizzle Kit CLI for migrations, schema push/pull, and database introspection
---

# Drizzle Kit

Drizzle Kit is the CLI companion for managing database schema migrations.

## Configuration

```ts
// drizzle.config.ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "postgresql",          // "postgresql" | "mysql" | "sqlite" | "turso" | "singlestore"
  schema: "./src/db/schema.ts",   // Path to schema file(s) or directory
  out: "./drizzle",               // Migration output directory (default: "./drizzle")
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
  // Optional
  strict: true,                   // Always ask for confirmation
  verbose: true,                  // Print SQL statements
  schemaFilter: ["public"],       // Filter schemas to manage
  tablesFilter: ["users", "posts"], // Filter specific tables
  migrations: {
    prefix: "timestamp",          // "timestamp" | "supabase" | "unix" | "none"
    table: "__drizzle_migrations__",
    schema: "public",
  },
});
```

### Multiple Config Files

```bash
npx drizzle-kit generate --config=drizzle-dev.config.ts
npx drizzle-kit generate --config=drizzle-prod.config.ts
```

## Commands

### `generate` - Create migration files

Compares your schema to the previous state and generates SQL migration files:

```bash
npx drizzle-kit generate
```

Output: `./drizzle/0001_migration_name.sql`

### `migrate` - Apply migrations

Runs pending migrations against the database:

```bash
npx drizzle-kit migrate
```

### Programmatic Migration

```ts
import { drizzle } from "drizzle-orm/node-postgres";
import { migrate } from "drizzle-orm/node-postgres/migrator";

const db = drizzle(process.env.DATABASE_URL!);
await migrate(db, { migrationsFolder: "./drizzle" });
```

### `push` - Push schema directly (no migration files)

Applies schema changes directly to the database. Useful for prototyping and development:

```bash
npx drizzle-kit push
```

### `pull` - Introspect database

Generates Drizzle schema from an existing database:

```bash
npx drizzle-kit pull
```

Outputs TypeScript schema files and migration snapshots.

### `check` - Validate migrations

Checks for collisions or inconsistencies in generated migrations:

```bash
npx drizzle-kit check
```

### `studio` - Visual database browser

Launches Drizzle Studio, a web-based database explorer:

```bash
npx drizzle-kit studio
```

### `up` - Upgrade snapshots

Upgrades Drizzle metadata snapshots when the format changes between versions:

```bash
npx drizzle-kit up
```

## Custom Migrations

Add custom SQL to generated migrations or create standalone migration files:

```bash
npx drizzle-kit generate --custom
```

This creates an empty migration file you can fill with custom SQL.

## Key Points

- Always export all schema objects (tables, enums, etc.) so drizzle-kit can discover them
- Use `push` for rapid prototyping, `generate` + `migrate` for production
- Migration files are versioned and should be committed to version control
- `pull` is useful for working with existing databases or reverse-engineering schemas
- The `drizzle` output directory contains both SQL files and JSON snapshots

<!--
Source references:
- https://orm.drizzle.team/docs/drizzle-config-file
- https://orm.drizzle.team/docs/kit-overview
- https://orm.drizzle.team/docs/drizzle-kit-generate
- https://orm.drizzle.team/docs/drizzle-kit-migrate
- https://orm.drizzle.team/docs/drizzle-kit-push
- https://orm.drizzle.team/docs/drizzle-kit-pull
- https://orm.drizzle.team/docs/kit-custom-migrations
-->
