---
name: features-transactions
description: Drizzle ORM transactions, savepoints, and isolation levels
---

# Transactions

## Basic Transaction

```ts
await db.transaction(async (tx) => {
  await tx.update(accounts).set({ balance: sql`${accounts.balance} - 100` }).where(eq(accounts.id, 1));
  await tx.update(accounts).set({ balance: sql`${accounts.balance} + 100` }).where(eq(accounts.id, 2));
});
```

## Rollback

```ts
await db.transaction(async (tx) => {
  const [account] = await tx.select({ balance: accounts.balance })
    .from(accounts).where(eq(accounts.id, 1));

  if (account.balance < 100) {
    tx.rollback(); // Throws an exception that rolls back the transaction
    return;        // Unreachable but satisfies TS
  }

  await tx.update(accounts).set({ balance: sql`${accounts.balance} - 100` }).where(eq(accounts.id, 1));
});
```

## Return Values

```ts
const newBalance = await db.transaction(async (tx) => {
  await tx.update(accounts).set({ balance: sql`${accounts.balance} - 100` }).where(eq(accounts.id, 1));
  const [account] = await tx.select({ balance: accounts.balance }).from(accounts).where(eq(accounts.id, 1));
  return account.balance;
});
```

## Nested Transactions (Savepoints)

```ts
await db.transaction(async (tx) => {
  await tx.update(accounts).set({ balance: sql`${accounts.balance} - 100` }).where(eq(accounts.id, 1));

  await tx.transaction(async (tx2) => {
    // This is a savepoint
    await tx2.update(users).set({ name: "Updated" }).where(eq(users.id, 1));
  });
});
```

## Relational Queries in Transactions

```ts
await db.transaction(async (tx) => {
  const users = await tx.query.users.findMany({
    with: { posts: true },
  });
});
```

## Dialect-Specific Options

### PostgreSQL

```ts
await db.transaction(async (tx) => { /* ... */ }, {
  isolationLevel: "read committed",  // "read uncommitted" | "read committed" | "repeatable read" | "serializable"
  accessMode: "read write",          // "read only" | "read write"
  deferrable: true,
});
```

### MySQL

```ts
await db.transaction(async (tx) => { /* ... */ }, {
  isolationLevel: "read committed",
  accessMode: "read write",
  withConsistentSnapshot: true,
});
```

### SQLite

```ts
await db.transaction(async (tx) => { /* ... */ }, {
  behavior: "deferred",  // "deferred" | "immediate" | "exclusive"
});
```

<!--
Source references:
- https://orm.drizzle.team/docs/transactions
-->
