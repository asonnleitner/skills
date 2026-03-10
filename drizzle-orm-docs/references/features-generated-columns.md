---
name: features-generated-columns
description: Generated (computed) columns and custom column types in Drizzle ORM
---

# Generated Columns

Columns whose values are computed from other columns. Support varies by database.

| Database | Virtual | Stored |
|----------|---------|--------|
| PostgreSQL | No | Yes |
| MySQL | Yes | Yes |
| SQLite | Yes | Yes |
| MSSQL | Yes (Virtual) | Yes (Persisted) |
| CockroachDB | No | Yes |

## PostgreSQL (Stored Only)

```typescript
export const test = pgTable("test", {
  name: text("first_name"),
  generatedName: text("gen_name").generatedAlwaysAs(
    (): SQL => sql`'hi, ' || ${test.name} || '!'`
  ),
});
// GENERATED ALWAYS AS ('hi, ' || "first_name" || '!') STORED
```

## MySQL (Stored & Virtual)

```typescript
export const users = mysqlTable("users", {
  name: text("name"),
  storedGen: text("stored_gen").generatedAlwaysAs(
    (): SQL => sql`concat(${users.name}, ' hello')`,
    { mode: "stored" }
  ),
  virtualGen: text("virtual_gen").generatedAlwaysAs(
    (): SQL => sql`concat(${users.name}, ' hello')`,
    { mode: "virtual" }  // default
  ),
});
```

## Full-Text Search with Generated Column (PostgreSQL)

```typescript
import { customType, index, integer, pgTable, text } from "drizzle-orm/pg-core";

const tsVector = customType<{ data: string }>({
  dataType() { return "tsvector"; },
});

export const test = pgTable("test", {
  id: integer("id").primaryKey().generatedAlwaysAsIdentity(),
  content: text("content"),
  contentSearch: tsVector("content_search").generatedAlwaysAs(
    (): SQL => sql`to_tsvector('english', ${test.content})`
  ),
}, (t) => [
  index("idx_content_search").using("gin", t.contentSearch),
]);
```

# Custom Column Types

Define reusable custom column types with `customType`:

```typescript
import { customType } from 'drizzle-orm/pg-core';

const customJsonb = <TData>(name: string) =>
  customType<{ data: TData; driverData: string }>({
    dataType() { return 'jsonb'; },
    toDriver(value: TData): string { return JSON.stringify(value); },
  })(name);

const customTimestamp = customType<{
  data: Date;
  driverData: string;
  config: { withTimezone: boolean; precision?: number };
}>({
  dataType(config) {
    const precision = config.precision !== undefined ? ` (${config.precision})` : '';
    return `timestamp${precision}${config.withTimezone ? ' with time zone' : ''}`;
  },
  fromDriver(value: string): Date { return new Date(value); },
});

// Usage
const table = pgTable('table', {
  data: customJsonb<string[]>('data'),
  createdAt: customTimestamp('created_at', { withTimezone: true })
    .notNull().default(sql`now()`),
});
```

<!--
Source references:
- https://orm.drizzle.team/docs/generated-columns
- https://orm.drizzle.team/docs/custom-types
-->
