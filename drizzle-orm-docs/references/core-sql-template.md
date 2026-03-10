---
name: core-sql-template
description: The sql template tag for raw SQL expressions, dynamic query building, and custom operators
---

# SQL Template Tag

The `sql` template literal from `drizzle-orm` allows embedding raw SQL with automatic parameterization and escaping.

## Basic Usage

```typescript
import { sql } from 'drizzle-orm';

// In select
const result = await db.select({
  id: users.id,
  fullName: sql<string>`${users.firstName} || ' ' || ${users.lastName}`,
}).from(users);

// In where
await db.select().from(users).where(sql`${users.id} = 42`);
await db.select().from(users).where(sql`lower(${users.name}) = 'aaron'`);

// In orderBy
await db.select().from(users).orderBy(sql`${users.id} desc`);
```

## Type Annotations

```typescript
// Specify return type with sql<T>
const result = await db.select({
  lowerName: sql<string>`lower(${users.name})`,
  count: sql<number>`cast(count(*) as int)`,
}).from(users);
```

## sql.mapWith()

Map driver values at runtime:

```typescript
const result = await db.select({
  visitsCount: sql`count(*)`.mapWith(Number),
}).from(users);
```

## sql.as() — Aliasing

Required when using `sql` in relational queries extras:

```typescript
const result = await db.query.users.findMany({
  extras: {
    lowerName: sql<string>`lower(${users.name})`.as('lower_name'),
  },
});
```

## sql.raw()

Insert raw SQL without any processing or escaping. Use carefully:

```typescript
sql`select * from ${sql.raw(tableName)}`;
```

## sql.join()

Join multiple SQL chunks with a separator:

```typescript
const columns = [users.id, users.name, users.email];
const query = sql`select ${sql.join(columns, sql.raw(', '))} from ${users}`;
```

## sql.empty() and sql.append()

Build SQL incrementally:

```typescript
const query = sql.empty();
query.append(sql`select * from ${users}`);
query.append(sql` where ${users.id} = 42`);
```

## sql.fromList()

Combine multiple SQL chunks:

```typescript
const chunks = [sql`select * from ${users}`, sql` where id = 42`];
const query = sql.fromList(chunks);
```

## Custom Filter Operators

```typescript
import { AnyColumn, sql } from 'drizzle-orm';

function lower(col: AnyColumn) {
  return sql`lower(${col})`;
}

function lenlt(col: AnyColumn, value: number) {
  return sql`length(${col}) < ${value}`;
}

await db.select().from(users).where(lenlt(users.name, 10));
```

## Converting to String

```typescript
import { PgDialect } from 'drizzle-orm/pg-core';
import { MySqlDialect } from 'drizzle-orm/mysql-core';
import { SQLiteDialect } from 'drizzle-orm/sqlite-core';

const pgDialect = new PgDialect();
const { sql: queryString, params } = pgDialect.sqlToQuery(sql`select * from ${users}`);
// queryString: 'select * from "users"'
```

## sql.placeholder() — Prepared Statements

```typescript
const prepared = db.select().from(users)
  .where(eq(users.id, sql.placeholder('id')))
  .prepare('get_user');

await prepared.execute({ id: 42 });
```

<!--
Source references:
- https://orm.drizzle.team/docs/sql
-->
