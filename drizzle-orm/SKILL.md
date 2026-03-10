---
name: drizzle-orm
description: Drizzle ORM - TypeScript SQL ORM with type-safe query builder, relational queries, and migration toolkit for PostgreSQL, MySQL, and SQLite. Use when working with drizzle-orm, drizzle-kit, or database schema/queries in TypeScript projects.
metadata:
  author: Andreas Sonnleitner
  version: "2026.3.10"
  source: Generated from https://github.com/drizzle-team/drizzle-orm-docs, scripts located at https://github.com/asonnleitner/skills
---

> The skill is based on drizzle-orm v0.45.1, generated at 2026-03-10.

Drizzle ORM is a lightweight, type-safe TypeScript ORM that maps closely to SQL. It supports PostgreSQL, MySQL, and SQLite with dialect-specific APIs. Drizzle Kit is the companion CLI for schema migrations.

## Core

| Topic | Description | Reference |
|-------|-------------|-----------|
| Schema Declaration | Tables, columns, enums, type inference across PG/MySQL/SQLite | [core-schema](references/core-schema.md) |
| SQL Queries | Select, insert, update, delete with type-safe operators | [core-queries](references/core-queries.md) |
| Joins & Set Operations | Join types, subqueries, union/intersect/except | [core-joins](references/core-joins.md) |
| Indexes & Constraints | Indexes, foreign keys, check constraints, views | [core-indexes-constraints](references/core-indexes-constraints.md) |

## Features

| Topic | Description | Reference |
|-------|-------------|-----------|
| Relations & Relational Queries | defineRelations, db.query API, findMany/findFirst with nested loading | [features-relations](references/features-relations.md) |
| Drizzle Kit | Migrations CLI - generate, migrate, push, pull, studio | [features-drizzle-kit](references/features-drizzle-kit.md) |
| Transactions | Transactions, savepoints, isolation levels | [features-transactions](references/features-transactions.md) |

## Advanced

| Topic | Description | Reference |
|-------|-------------|-----------|
| Advanced Features | Dynamic queries, batch API, read replicas, custom types, RLS | [advanced-features](references/advanced-features.md) |

## Best Practices

| Topic | Description | Reference |
|-------|-------------|-----------|
| Common Patterns | Pagination, upserts, full-text search, vector search, Zod validation, seeding | [best-practices-patterns](references/best-practices-patterns.md) |

## Quick Reference

### Imports

```ts
// Schema (pick your dialect)
import { pgTable, integer, text, varchar, boolean, timestamp, uuid, jsonb, pgEnum } from "drizzle-orm/pg-core";
import { mysqlTable, int, varchar, text, boolean, timestamp } from "drizzle-orm/mysql-core";
import { sqliteTable, integer, text } from "drizzle-orm/sqlite-core";

// Operators & utilities
import { eq, ne, gt, gte, lt, lte, and, or, not, like, ilike, inArray, between, isNull, sql, desc, asc } from "drizzle-orm";

// Relations
import { defineRelations } from "drizzle-orm";

// Drizzle Kit config
import { defineConfig } from "drizzle-kit";
```

### CLI Commands

```bash
npx drizzle-kit generate   # Generate migration SQL files
npx drizzle-kit migrate    # Apply pending migrations
npx drizzle-kit push       # Push schema directly (dev/prototyping)
npx drizzle-kit pull       # Introspect existing database
npx drizzle-kit studio     # Launch visual database browser
```
