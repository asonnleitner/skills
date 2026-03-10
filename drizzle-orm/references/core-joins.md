---
name: core-joins
description: Drizzle ORM join types, subqueries, and set operations
---

# Joins & Set Operations

## Join Types

Drizzle supports `leftJoin`, `rightJoin`, `innerJoin`, `fullJoin`, `crossJoin`, and lateral variants.

```ts
import { eq } from "drizzle-orm";

// Left join - right side nullable
const result = await db.select().from(users)
  .leftJoin(pets, eq(users.id, pets.ownerId));
// Type: { users: { id, name }, pets: { id, name, ownerId } | null }[]

// Inner join - both sides required
const result = await db.select().from(users)
  .innerJoin(pets, eq(users.id, pets.ownerId));
// Type: { users: { id, name }, pets: { id, name, ownerId } }[]

// Right join - left side nullable
const result = await db.select().from(users)
  .rightJoin(pets, eq(users.id, pets.ownerId));

// Full join - both sides nullable
const result = await db.select().from(users)
  .fullJoin(pets, eq(users.id, pets.ownerId));

// Cross join
const result = await db.select().from(users).crossJoin(pets);
```

### Lateral Joins (PostgreSQL)

Lateral joins allow subqueries to reference columns from preceding tables:

```ts
const subquery = db.select().from(pets).where(gte(users.age, 16)).as("userPets");
const result = await db.select().from(users)
  .leftJoinLateral(subquery, sql`true`);
```

### Partial Select with Joins

```ts
const result = await db.select({
  userId: users.id,
  petName: pets.name,
}).from(users)
  .leftJoin(pets, eq(users.id, pets.ownerId));
```

### Multiple Joins

```ts
const result = await db.select()
  .from(users)
  .leftJoin(pets, eq(users.id, pets.ownerId))
  .leftJoin(posts, eq(users.id, posts.authorId));
```

### Join with Aliases (self-join)

```ts
import { alias } from "drizzle-orm";

const parent = alias(users, "parent");
const result = await db.select()
  .from(users)
  .leftJoin(parent, eq(users.parentId, parent.id));
```

## Subqueries

```ts
// As a source in FROM
const sq = db.select({
  userId: users.id,
  count: sql<number>`count(${posts.id})`.as("post_count"),
}).from(users)
  .leftJoin(posts, eq(users.id, posts.authorId))
  .groupBy(users.id)
  .as("sq");

await db.select().from(sq).where(gt(sq.count, 5));

// In WHERE with exists
await db.select().from(users).where(
  exists(db.select().from(posts).where(eq(posts.authorId, users.id)))
);
```

## Set Operations

```ts
// Union
await db.select().from(users).union(db.select().from(archivedUsers));

// Union all (keeps duplicates)
await db.select().from(users).unionAll(db.select().from(archivedUsers));

// Intersect
await db.select().from(users).intersect(db.select().from(premiumUsers));

// Except
await db.select().from(users).except(db.select().from(bannedUsers));
```

<!--
Source references:
- https://orm.drizzle.team/docs/joins
- https://orm.drizzle.team/docs/set-operations
-->
