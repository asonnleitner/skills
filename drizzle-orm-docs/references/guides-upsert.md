---
name: guides-upsert
description: Upsert patterns for PostgreSQL, MySQL, and SQLite with bulk operations
---

# Upsert Patterns

## PostgreSQL / SQLite

```typescript
// Basic upsert
await db.insert(users)
  .values({ id: 1, name: 'John' })
  .onConflictDoUpdate({
    target: users.id,
    set: { name: 'Super John' },
  });
```

### Bulk Upsert with Excluded Values

Use `sql.raw('excluded.column_name')` to reference the attempted insert values:

```typescript
const values = [
  { id: 1, lastLogin: new Date() },
  { id: 2, lastLogin: new Date(Date.now() + 3600000) },
];

await db.insert(users)
  .values(values)
  .onConflictDoUpdate({
    target: users.id,
    set: { lastLogin: sql.raw(`excluded.${users.lastLogin.name}`) },
  });
```

### Reusable Conflict Update Builder

```typescript
import { SQL, getColumns, sql } from 'drizzle-orm';
import { PgTable } from 'drizzle-orm/pg-core';

const buildConflictUpdateColumns = <
  T extends PgTable,
  Q extends keyof T['_']['columns']
>(table: T, columns: Q[]) => {
  const cls = getColumns(table);
  return columns.reduce((acc, column) => {
    acc[column] = sql.raw(`excluded.${cls[column].name}`);
    return acc;
  }, {} as Record<Q, SQL>);
};

await db.insert(users)
  .values(values)
  .onConflictDoUpdate({
    target: users.id,
    set: buildConflictUpdateColumns(users, ['lastLogin', 'active']),
  });
```

### Composite Key Upsert

```typescript
await db.insert(inventory)
  .values({ warehouseId: 1, productId: 1, quantity: 100 })
  .onConflictDoUpdate({
    target: [inventory.warehouseId, inventory.productId],
    set: { quantity: sql`${inventory.quantity} + 100` },
  });
```

### Conditional Upsert with setWhere

Only update if values actually changed:

```typescript
await db.insert(products)
  .values(data)
  .onConflictDoUpdate({
    target: products.id,
    set: { price: sql.raw(`excluded.${products.price.name}`) },
    setWhere: sql`${products.price} != excluded.${sql.raw(products.price.name)}`,
  });
```

### Keep Column Unchanged

Reference the existing column value to skip updating it:

```typescript
await db.insert(users)
  .values(data)
  .onConflictDoUpdate({
    target: users.id,
    set: { ...data, email: sql`${users.email}` }, // email stays unchanged
  });
```

## MySQL

MySQL uses `onDuplicateKeyUpdate` (auto-detects target from primary/unique keys):

```typescript
await db.insert(users)
  .values({ id: 1, name: 'John' })
  .onDuplicateKeyUpdate({ set: { name: 'Super John' } });

// Bulk with values()
await db.insert(users).values(values)
  .onDuplicateKeyUpdate({
    set: { lastLogin: sql`values(${users.lastLogin})` },
  });
```

<!--
Source references:
- https://orm.drizzle.team/docs/guides/upsert
-->
