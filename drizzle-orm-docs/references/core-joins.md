---
name: core-joins
description: JOIN operations in Drizzle ORM with type-safe nullability and aggregation patterns
---

# Joins

Drizzle provides type-safe joins where nullability is automatically inferred.

## Join Types

```typescript
// LEFT JOIN — right side nullable
const result = await db.select().from(users).leftJoin(pets, eq(users.id, pets.ownerId));
// { user: { id, name }, pets: { id, name, ownerId } | null }[]

// RIGHT JOIN — left side nullable
const result = await db.select().from(users).rightJoin(pets, eq(users.id, pets.ownerId));
// { user: { id, name } | null, pets: { id, name, ownerId } }[]

// INNER JOIN — neither side nullable
const result = await db.select().from(users).innerJoin(pets, eq(users.id, pets.ownerId));
// { user: { id, name }, pets: { id, name, ownerId } }[]

// FULL JOIN — both sides nullable
const result = await db.select().from(users).fullJoin(pets, eq(users.id, pets.ownerId));
// { user: { id, name } | null, pets: { id, name, ownerId } | null }[]

// CROSS JOIN
const result = await db.select().from(users).crossJoin(pets);
```

## Lateral Joins (PostgreSQL)

```typescript
const subquery = db.select().from(pets).where(gte(users.age, 16)).as('userPets');
const result = await db.select().from(users).leftJoinLateral(subquery, sql`true`);
```

## Partial Select in Joins

```typescript
await db.select({
  userId: users.id,
  petId: pets.id,
}).from(users).leftJoin(pets, eq(users.id, pets.ownerId));
// { userId: number, petId: number | null }[]
```

## Nested Select Objects

```typescript
await db.select({
  userId: users.id,
  pet: {
    id: pets.id,
    name: pets.name,
  },
}).from(users).fullJoin(pets, eq(users.id, pets.ownerId));
// { userId: number | null, pet: { id, name } | null }[]
```

## Aliases & Self-Joins

```typescript
import { alias } from "drizzle-orm";

const parent = alias(users, "parent");
const result = await db.select().from(users)
  .leftJoin(parent, eq(parent.id, users.parentId));
```

## Aggregating Join Results (One-to-Many)

```typescript
type User = typeof users.$inferSelect;
type Pet = typeof pets.$inferSelect;

const rows = await db.select({ user: users, pet: pets })
  .from(users).leftJoin(pets, eq(users.id, pets.ownerId));

const result = rows.reduce<Record<number, { user: User; pets: Pet[] }>>(
  (acc, row) => {
    if (!acc[row.user.id]) acc[row.user.id] = { user: row.user, pets: [] };
    if (row.pet) acc[row.user.id].pets.push(row.pet);
    return acc;
  }, {}
);
```

## Many-to-Many

```typescript
const usersToChatGroups = sqliteTable('users_to_chat_groups', {
  userId: integer('user_id').notNull().references(() => users.id),
  groupId: integer('group_id').notNull().references(() => chatGroups.id),
});

db.select()
  .from(usersToChatGroups)
  .leftJoin(users, eq(usersToChatGroups.userId, users.id))
  .leftJoin(chatGroups, eq(usersToChatGroups.groupId, chatGroups.id))
  .where(eq(chatGroups.id, 1));
```

<!--
Source references:
- https://orm.drizzle.team/docs/joins
-->
