---
name: features-relations
description: Drizzle ORM relational queries - defineRelations, findMany, findFirst with nested data loading
---

# Relations & Relational Queries

Drizzle relations provide a declarative way to query nested relational data without manual joins.

## Defining Relations (v2 - beta.1+)

```ts
import { defineRelations } from "drizzle-orm";
import * as p from "drizzle-orm/pg-core";

export const users = p.pgTable("users", {
  id: p.integer().primaryKey(),
  name: p.text().notNull(),
});

export const posts = p.pgTable("posts", {
  id: p.integer().primaryKey(),
  content: p.text().notNull(),
  authorId: p.integer("author_id").references(() => users.id),
});

export const comments = p.pgTable("comments", {
  id: p.integer().primaryKey(),
  text: p.text().notNull(),
  postId: p.integer("post_id").references(() => posts.id),
  authorId: p.integer("author_id").references(() => users.id),
});

export const relations = defineRelations({ users, posts, comments }, (r) => ({
  users: {
    posts: r.many.posts({
      from: r.users.id,
      to: r.posts.authorId,
    }),
  },
  posts: {
    author: r.one.users({
      from: r.posts.authorId,
      to: r.users.id,
    }),
    comments: r.many.comments({
      from: r.posts.id,
      to: r.comments.postId,
    }),
  },
  comments: {
    post: r.one.posts({
      from: r.comments.postId,
      to: r.posts.id,
    }),
    author: r.one.users({
      from: r.comments.authorId,
      to: r.users.id,
    }),
  },
}));
```

### Relation Options

- `from` / `to` - columns defining the relationship
- `optional: false` - makes the relation non-nullable in TypeScript types
- `alias` - required when multiple relations exist between same two tables
- `where` - filter condition for polymorphic relations

## Many-to-Many (through table)

```ts
const usersToGroups = p.pgTable("users_to_groups", {
  userId: p.integer("user_id").references(() => users.id),
  groupId: p.integer("group_id").references(() => groups.id),
}, (t) => [p.primaryKey(t.userId, t.groupId)]);

const relations = defineRelations({ users, groups, usersToGroups }, (r) => ({
  users: {
    groups: r.many.groups({
      from: r.users.id.through(r.usersToGroups.userId),
      to: r.groups.id.through(r.usersToGroups.groupId),
    }),
  },
  groups: {
    users: r.many.users({
      from: r.groups.id.through(r.usersToGroups.groupId),
      to: r.users.id.through(r.usersToGroups.userId),
    }),
  },
}));
```

## Initialize with Relations

```ts
import { drizzle } from "drizzle-orm/node-postgres";
import { relations } from "./relations";

const db = drizzle({ client: pool, relations });
```

## Querying (db.query API)

### findMany

```ts
// Basic
const allUsers = await db.query.users.findMany();

// With relations
const usersWithPosts = await db.query.users.findMany({
  with: {
    posts: true,
  },
});

// Nested relations
const result = await db.query.users.findMany({
  with: {
    posts: {
      with: {
        comments: true,
      },
    },
  },
});

// Select specific columns
const result = await db.query.users.findMany({
  columns: {
    id: true,
    name: true,
  },
  with: {
    posts: {
      columns: {
        content: true,
      },
    },
  },
});

// Exclude columns
const result = await db.query.users.findMany({
  columns: {
    password: false,
  },
});

// With filters and ordering
const result = await db.query.users.findMany({
  where: {
    role: "admin",
  },
  orderBy: {
    createdAt: "desc",
  },
  limit: 10,
  offset: 0,
});

// Filter with operators
import { gt } from "drizzle-orm";
const result = await db.query.users.findMany({
  where: (fields, ops) => ops.gt(fields.age, 18),
});

// Extras - computed/virtual fields
const result = await db.query.users.findMany({
  extras: {
    lowerName: sql<string>`lower(${users.name})`.as("lower_name"),
  },
});
```

### findFirst

```ts
// Adds LIMIT 1 automatically
const user = await db.query.users.findFirst({
  where: {
    id: 1,
  },
  with: {
    posts: true,
  },
});
```

## Key Points

- Relations are "soft" - they don't create foreign key constraints, just define query patterns
- `r.one.*` returns a single object; `r.many.*` returns an array
- The `db.query` API only works when `relations` are passed to `drizzle()`
- Use `with` to eagerly load related data (generates efficient SQL JOINs or lateral joins)
- Use `columns` for partial field selection
- Relational queries work inside transactions too

<!--
Source references:
- https://orm.drizzle.team/docs/relations-v2
- https://orm.drizzle.team/docs/rqb-v2
- https://orm.drizzle.team/docs/relations-schema-declaration
-->
