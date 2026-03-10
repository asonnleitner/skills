---
name: core-column-types
description: Column types for PostgreSQL, MySQL, and SQLite in Drizzle ORM
---

# Column Types

## PostgreSQL Column Types

Import from `drizzle-orm/pg-core`.

**Numeric:** `integer()`, `smallint()`, `bigint()`, `serial()`, `smallserial()`, `bigserial()`, `numeric({ precision, scale })`, `real()`, `doublePrecision()`

**String:** `text()`, `varchar({ length })`, `char({ length })`

**Boolean:** `boolean()`

**JSON:** `json()`, `jsonb()` — use `.$type<T>()` for type inference

**UUID:** `uuid()` — use `.defaultRandom()` for auto-generation

**Date/Time:** `timestamp({ withTimezone, precision, mode })`, `date({ mode })`, `time({ withTimezone, precision })`, `interval({ fields, precision })`

**Binary:** `bytea()`

**Geometric:** `point({ mode })`, `line({ mode })`

**Enum:**
```typescript
import { pgEnum } from "drizzle-orm/pg-core";
export const roleEnum = pgEnum("role", ["admin", "user", "guest"]);
// Usage: role: roleEnum().default("guest")
```

**Array types:**
```typescript
tags: text('tags').array(),        // text[]
matrix: integer('matrix').array().array(), // integer[][]
```

**Bigint modes:**
```typescript
bigint({ mode: 'number' })  // JS number (loses precision > 2^53)
bigint({ mode: 'bigint' })  // JS BigInt
```

**Timestamp modes:**
```typescript
timestamp({ mode: 'date' })    // JS Date object (default)
timestamp({ mode: 'string' })  // ISO string from driver
```

## MySQL Column Types

Import from `drizzle-orm/mysql-core`.

**Numeric:** `int()`, `tinyint()`, `smallint()`, `mediumint()`, `bigint({ mode })`, `float()`, `double()`, `decimal({ precision, scale })`, `real()`, `serial()`

**String:** `varchar({ length })`, `char({ length })`, `text()`, `tinytext()`, `mediumtext()`, `longtext()`

**Binary:** `binary({ length })`, `varbinary({ length })`, `blob()`, `tinyblob()`, `mediumblob()`, `longblob()`

**Boolean:** `boolean()` (tinyint(1) under the hood)

**Date/Time:** `date()`, `datetime({ mode, fsp })`, `timestamp({ mode, fsp })`, `time({ fsp })`, `year()`

**JSON:** `json()` — use `.$type<T>()` for type inference

**Enum:**
```typescript
import { mysqlEnum } from "drizzle-orm/mysql-core";
role: mysqlEnum(["admin", "user", "guest"]),
```

## SQLite Column Types

Import from `drizzle-orm/sqlite-core`. SQLite has 5 storage classes: NULL, INTEGER, REAL, TEXT, BLOB.

**Integer:** `integer()` — supports modes:
```typescript
integer()                          // number (default)
integer({ mode: 'boolean' })      // boolean (0/1)
integer({ mode: 'timestamp' })    // Date (unix seconds)
integer({ mode: 'timestamp_ms' }) // Date (unix milliseconds)
```

**Real:** `real()`

**Text:** `text()` — supports modes:
```typescript
text()                     // string
text({ mode: 'json' })    // JSON parsed
text({ enum: ['a', 'b'] }) // union type
```

**Blob:** `blob()` — supports modes:
```typescript
blob()                     // Buffer
blob({ mode: 'bigint' })  // BigInt
blob({ mode: 'json' })    // JSON
```

**Numeric:** `numeric()` — stores as text but with numeric affinity

## Common Column Modifiers

Available across all dialects:

```typescript
column.notNull()                      // NOT NULL
column.default(value)                 // Static default
column.default(sql`expression`)       // SQL expression default
column.$defaultFn(() => value)        // Runtime default (JS-side)
column.$onUpdateFn(() => value)       // Runtime on-update (JS-side)
column.$type<CustomType>()            // Override TypeScript type
column.primaryKey()                   // PRIMARY KEY
column.unique()                       // UNIQUE constraint
column.references(() => other.id)     // FOREIGN KEY
column.generatedAlwaysAs(expression)  // Generated column
```

<!--
Source references:
- https://orm.drizzle.team/docs/column-types/pg
- https://orm.drizzle.team/docs/column-types/mysql
- https://orm.drizzle.team/docs/column-types/sqlite
-->
