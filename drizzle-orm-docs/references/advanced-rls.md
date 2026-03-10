---
name: advanced-rls
description: Row-Level Security (RLS) policies in Drizzle ORM for PostgreSQL
---

# Row-Level Security (RLS)

PostgreSQL-only feature for controlling row access at the database level.

## Defining Roles & Policies

```typescript
import { pgTable, pgPolicy, pgRole, integer, text } from 'drizzle-orm/pg-core';
import { sql } from 'drizzle-orm';

// Define a role
export const admin = pgRole('admin');

// Table with RLS policies
export const users = pgTable('users', {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  name: text().notNull(),
  role: text().$type<'admin' | 'user'>().notNull(),
}, (table) => [
  // Admin can do anything
  pgPolicy('admin_all', {
    for: 'all',
    to: admin,
    using: sql`true`,
    withCheck: sql`true`,
  }),
  // Users can only read their own rows
  pgPolicy('user_select', {
    for: 'select',
    using: sql`${table.role} = current_setting('app.role')`,
  }),
]);
```

## Drizzle Kit Configuration

Enable RLS role management in `drizzle.config.ts`:

```typescript
export default defineConfig({
  dialect: 'postgresql',
  schema: './src/db/schema.ts',
  dbCredentials: { url: process.env.DATABASE_URL! },
  entities: {
    roles: {
      provider: '',  // empty string = Drizzle manages roles
      // Or exclude specific roles from management:
      // exclude: ['postgres', 'admin']
    },
  },
});
```

## Enable RLS on a Table

```typescript
import { pgTable } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  // columns...
}).enableRLS();
```

## Neon Integration

```typescript
import { crudPolicy, authenticatedRole, authUid } from 'drizzle-orm/neon';

export const users = pgTable('users', {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  userId: text().notNull(),
  name: text().notNull(),
}, (table) => [
  crudPolicy({
    role: authenticatedRole,
    read: authUid(table.userId),
    modify: authUid(table.userId),
  }),
]);
```

## Supabase Integration

```typescript
import { authenticatedRole, anonRole, authUid, supabaseAuthAdminRole } from 'drizzle-orm/supabase';

export const users = pgTable('users', {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  userId: text().notNull(),
}, (table) => [
  pgPolicy('authenticated_read', {
    for: 'select',
    to: authenticatedRole,
    using: sql`(select auth.uid()) = ${table.userId}`,
  }),
]);
```

<!--
Source references:
- https://orm.drizzle.team/docs/rls
-->
