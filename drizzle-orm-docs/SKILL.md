---
name: drizzle-orm
description: TypeScript ORM for SQL databases with type-safe queries, schema declaration, migrations, and relational query builder
metadata:
  author: Andreas Sonnleitner
  version: "2026.3.10"
  source: Generated from https://github.com/drizzle-team/drizzle-orm-docs, scripts located at https://github.com/asonnleitner/skills
---

> The skill is based on Drizzle ORM v0.45.1, generated at 2026-03-10.

Drizzle ORM is a TypeScript ORM for SQL databases that provides type-safe query building, schema declaration in TypeScript, automatic migrations, and a relational query builder. It supports PostgreSQL, MySQL, SQLite, MSSQL, SingleStore, and CockroachDB with a thin abstraction layer that maps closely to SQL.

## Core

| Topic | Description | Reference |
|-------|-------------|-----------|
| Schema Declaration | Table definitions, column naming, casing, reusable patterns | [core-schema-declaration](references/core-schema-declaration.md) |
| Column Types | PostgreSQL, MySQL, and SQLite column types and modifiers | [core-column-types](references/core-column-types.md) |
| Indexes & Constraints | Primary keys, foreign keys, unique, check, indexes | [core-indexes-constraints](references/core-indexes-constraints.md) |
| Select Queries | Filters, ordering, pagination, aggregations, CTEs, subqueries | [core-select](references/core-select.md) |
| Insert | Insert, returning, upsert, INSERT...SELECT | [core-insert](references/core-insert.md) |
| Update | Update, returning, UPDATE...FROM, increment/decrement | [core-update](references/core-update.md) |
| Delete | Delete, returning, CTEs | [core-delete](references/core-delete.md) |
| Joins | LEFT/RIGHT/INNER/FULL/CROSS joins, lateral, aliases, aggregation | [core-joins](references/core-joins.md) |
| Operators | Comparison, logical, array, pattern matching, set operations | [core-operators](references/core-operators.md) |
| SQL Template | Raw SQL expressions, sql.join, sql.raw, placeholders | [core-sql-template](references/core-sql-template.md) |

## Features

| Topic | Description | Reference |
|-------|-------------|-----------|
| Relations | One-to-one, one-to-many, many-to-many relation definitions | [features-relations](references/features-relations.md) |
| Relational Queries | db.query API with nested relations, filters, pagination | [features-relational-queries](references/features-relational-queries.md) |
| Transactions | Savepoints, rollbacks, dialect-specific isolation levels | [features-transactions](references/features-transactions.md) |
| Views | Regular views, materialized views, existing views | [features-views](references/features-views.md) |
| Schemas | Database schema namespacing (PostgreSQL, MySQL, etc.) | [features-schemas](references/features-schemas.md) |
| Dynamic Queries | $dynamic() for reusable query modifier functions | [features-dynamic-queries](references/features-dynamic-queries.md) |
| Generated Columns | Computed columns and custom column types | [features-generated-columns](references/features-generated-columns.md) |
| Connections | Database driver connection patterns for all supported drivers | [features-connections](references/features-connections.md) |

## Drizzle Kit

| Topic | Description | Reference |
|-------|-------------|-----------|
| Kit Overview | CLI commands, configuration file, driver configs | [drizzle-kit-overview](references/drizzle-kit-overview.md) |
| Migrations | Generate, apply, push, pull, custom migrations, runtime migrations | [drizzle-kit-migrations](references/drizzle-kit-migrations.md) |

## Advanced

| Topic | Description | Reference |
|-------|-------------|-----------|
| Performance | Prepared statements, serverless tips, read replicas, batch API | [advanced-performance](references/advanced-performance.md) |
| Row-Level Security | RLS policies, roles, Neon/Supabase integrations | [advanced-rls](references/advanced-rls.md) |
| Seeding | Deterministic data generation with drizzle-seed | [advanced-seeding](references/advanced-seeding.md) |
| Caching | Upstash Redis caching, custom cache implementations | [advanced-caching](references/advanced-caching.md) |

## Guides

| Topic | Description | Reference |
|-------|-------------|-----------|
| Pagination | Limit/offset and cursor-based pagination patterns | [guides-pagination](references/guides-pagination.md) |
| Upsert | Conflict handling patterns for PostgreSQL, MySQL, SQLite | [guides-upsert](references/guides-upsert.md) |
| Conditional Filters | Dynamic WHERE clauses, filter arrays, custom operators | [guides-conditional-filters](references/guides-conditional-filters.md) |
| Common Patterns | Counting, column selection, timestamps, unique emails | [guides-common-patterns](references/guides-common-patterns.md) |
| Full-Text & Vector Search | PostgreSQL full-text search and pgvector similarity | [guides-full-text-search](references/guides-full-text-search.md) |

## Integrations

| Topic | Description | Reference |
|-------|-------------|-----------|
| Schema Validation | Zod, Valibot, Arktype, TypeBox, Effect schema generation | [integrations-schema-validation](references/integrations-schema-validation.md) |
| GraphQL, Prisma & ESLint | GraphQL server, Prisma extension, ESLint plugin | [integrations-graphql-prisma](references/integrations-graphql-prisma.md) |
