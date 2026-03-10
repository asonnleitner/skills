---
name: features-relations
description: Drizzle ORM relations API for defining one-to-one, one-to-many, and many-to-many relationships
---

# Relations

Drizzle relations are application-level declarations used by the relational query builder. They do not create foreign keys in the database — use `.references()` for that.

## One-to-One

```typescript
import { pgTable, serial, text, integer } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name'),
});

export const profiles = pgTable('profiles', {
  id: serial('id').primaryKey(),
  bio: text('bio'),
  userId: integer('user_id').notNull().references(() => users.id),
});

export const usersRelations = relations(users, ({ one }) => ({
  profile: one(profiles, { fields: [users.id], references: [profiles.userId] }),
}));

export const profilesRelations = relations(profiles, ({ one }) => ({
  user: one(users, { fields: [profiles.userId], references: [users.id] }),
}));
```

## One-to-Many

```typescript
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name'),
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  content: text('content'),
  authorId: integer('author_id').notNull().references(() => users.id),
});

export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

export const postsRelations = relations(posts, ({ one }) => ({
  author: one(users, { fields: [posts.authorId], references: [users.id] }),
}));
```

## Many-to-Many

Use a junction table:

```typescript
export const users = pgTable('users', { id: serial('id').primaryKey() });
export const groups = pgTable('groups', { id: serial('id').primaryKey() });

export const usersToGroups = pgTable('users_to_groups', {
  userId: integer('user_id').notNull().references(() => users.id),
  groupId: integer('group_id').notNull().references(() => groups.id),
}, (t) => [primaryKey({ columns: [t.userId, t.groupId] })]);

export const usersRelations = relations(users, ({ many }) => ({
  usersToGroups: many(usersToGroups),
}));

export const groupsRelations = relations(groups, ({ many }) => ({
  usersToGroups: many(usersToGroups),
}));

export const usersToGroupsRelations = relations(usersToGroups, ({ one }) => ({
  user: one(users, { fields: [usersToGroups.userId], references: [users.id] }),
  group: one(groups, { fields: [usersToGroups.groupId], references: [groups.id] }),
}));
```

## Disambiguating Relations

When multiple relations exist between the same tables, use `relationName`:

```typescript
export const usersRelations = relations(users, ({ one }) => ({
  invitee: one(users, { fields: [users.inviteeId], references: [users.id], relationName: 'invitee' }),
}));
```

## Foreign Key Actions

Foreign keys (not relations) support cascade actions:

```typescript
userId: integer('user_id').references(() => users.id, {
  onDelete: 'cascade',  // cascade | no action | restrict | set default | set null
  onUpdate: 'no action',
}),
```

## Relations v2 (Beta)

The new API uses `defineRelations` separately from schema:

```typescript
import { defineRelations } from 'drizzle-orm';

export const relations = defineRelations(schema, (r) => ({
  users: {
    posts: r.many.posts(),
    profile: r.one.profiles({
      from: r.users.id,
      to: r.profiles.userId,
    }),
  },
  posts: {
    author: r.one.users({
      from: r.posts.authorId,
      to: r.users.id,
    }),
  },
}));
```

<!--
Source references:
- https://orm.drizzle.team/docs/relations
- https://orm.drizzle.team/docs/relations-v2
-->
