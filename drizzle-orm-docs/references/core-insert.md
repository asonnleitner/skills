---
name: core-insert
description: INSERT operations in Drizzle ORM including returning, upserts, and INSERT...SELECT
---

# Insert

## Basic Insert

```typescript
await db.insert(users).values({ name: 'Andrew' });

// Multiple rows
await db.insert(users).values([{ name: 'Andrew' }, { name: 'Dan' }]);
```

## Type Inference

```typescript
type NewUser = typeof users.$inferInsert;

const insertUser = async (user: NewUser) => {
  return db.insert(users).values(user);
};
```

## Returning (PostgreSQL, SQLite, CockroachDB)

```typescript
await db.insert(users).values({ name: "Dan" }).returning();
await db.insert(users).values({ name: "Dan" }).returning({ insertedId: users.id });
```

## $returningId (MySQL, SingleStore)

```typescript
const result = await db.insert(users).values([{ name: 'John' }, { name: 'Jane' }]).$returningId();
// ^? { id: number }[]
```

## Upsert — On Conflict Do Update (PostgreSQL, SQLite)

```typescript
await db.insert(users)
  .values({ id: 1, name: 'Dan' })
  .onConflictDoUpdate({ target: users.id, set: { name: 'John' } });

// Do nothing on conflict
await db.insert(users)
  .values({ id: 1, name: 'John' })
  .onConflictDoNothing({ target: users.id });

// Composite key target
await db.insert(users)
  .values({ firstName: 'John', lastName: 'Doe' })
  .onConflictDoUpdate({
    target: [users.firstName, users.lastName],
    set: { firstName: 'John1' },
  });

// With where clauses
await db.insert(employees)
  .values({ employeeId: 123, name: 'John Doe' })
  .onConflictDoUpdate({
    target: employees.employeeId,
    set: { name: sql`excluded.name` },
    setWhere: sql`name <> 'John Doe'`,
  });
```

## Upsert — On Duplicate Key Update (MySQL)

```typescript
await db.insert(users)
  .values({ id: 1, name: 'John' })
  .onDuplicateKeyUpdate({ set: { name: 'John' } });

// No-op upsert (useful for INSERT IGNORE equivalent)
await db.insert(users)
  .values({ id: 1, name: 'John' })
  .onDuplicateKeyUpdate({ set: { id: sql`id` } });
```

## INSERT INTO ... SELECT

```typescript
// From query builder
await db.insert(employees).select(
  db.select({ name: users.name }).from(users).where(eq(users.role, 'employee'))
);

// From callback
await db.insert(employees).select(
  (qb) => qb.select({ name: users.name }).from(users).where(eq(users.role, 'employee'))
);
```

## WITH Insert

```typescript
const userCount = db.$with('user_count').as(
  db.select({ value: sql`count(*)`.as('value') }).from(users)
);

const result = await db.with(userCount)
  .insert(users)
  .values([{ username: 'user1', admin: sql`((select * from ${userCount}) = 0)` }])
  .returning({ admin: users.admin });
```

<!--
Source references:
- https://orm.drizzle.team/docs/insert
-->
