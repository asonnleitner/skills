---
name: features-transactions
description: Database transactions in Drizzle ORM with savepoints, rollbacks, and dialect-specific config
---

# Transactions

## Basic Transaction

```typescript
await db.transaction(async (tx) => {
  await tx.insert(users).values({ name: 'John' });
  await tx.insert(posts).values({ title: 'Hello', authorId: 1 });
});
```

## Returning Values

```typescript
const result = await db.transaction(async (tx) => {
  const [user] = await tx.insert(users).values({ name: 'John' }).returning();
  await tx.insert(posts).values({ title: 'Hello', authorId: user.id });
  return user;
});
```

## Nested Transactions (Savepoints)

```typescript
await db.transaction(async (tx) => {
  await tx.insert(users).values({ name: 'John' });

  await tx.transaction(async (nestedTx) => {
    await nestedTx.insert(posts).values({ title: 'Hello' });
    // Creates a SAVEPOINT
  });
});
```

## Manual Rollback

```typescript
await db.transaction(async (tx) => {
  const [user] = await tx.insert(users).values({ name: 'John' }).returning();

  if (someCondition) {
    tx.rollback(); // Rolls back the entire transaction
    // Code after rollback() won't execute
  }

  await tx.insert(posts).values({ title: 'Hello', authorId: user.id });
});
```

## Relational Queries in Transactions

```typescript
await db.transaction(async (tx) => {
  const user = await tx.query.users.findFirst({
    where: eq(users.id, 1),
    with: { posts: true },
  });
});
```

## Dialect-Specific Configuration

**PostgreSQL:**
```typescript
await db.transaction(async (tx) => { /* ... */ }, {
  isolationLevel: 'read committed', // read uncommitted | read committed | repeatable read | serializable
  accessMode: 'read write',         // read only | read write
  deferrable: true,
});
```

**MySQL:**
```typescript
await db.transaction(async (tx) => { /* ... */ }, {
  isolationLevel: 'read committed',
  accessMode: 'read write',
  withConsistentSnapshot: true,
});
```

**SQLite:**
```typescript
await db.transaction(async (tx) => { /* ... */ }, {
  behavior: 'deferred', // deferred | immediate | exclusive
});
```

<!--
Source references:
- https://orm.drizzle.team/docs/transactions
-->
