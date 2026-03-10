---
name: integrations-graphql-prisma
description: GraphQL server generation and Prisma extension for Drizzle ORM
---

# GraphQL Integration

Generate a GraphQL server from your Drizzle schema:

```typescript
import { buildSchema } from 'drizzle-graphql';
import { drizzle } from 'drizzle-orm/node-postgres';
import * as dbSchema from './schema';

const db = drizzle({ client, schema: dbSchema });
const { schema } = buildSchema(db);
```

**With Apollo Server:**
```typescript
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';

const server = new ApolloServer({ schema });
const { url } = await startStandaloneServer(server);
```

**With GraphQL Yoga:**
```typescript
import { createYoga } from 'graphql-yoga';
import { createServer } from 'node:http';

const yoga = createYoga({ schema });
const server = createServer(yoga);
server.listen(4000);
```

**Custom queries/mutations:**
```typescript
const { entities } = buildSchema(db);

const customSchema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: 'Query',
    fields: {
      users: entities.queries.users,         // Generated query
      customUsers: {                          // Custom query
        type: new GraphQLList(new GraphQLNonNull(entities.types.UsersItem)),
        args: { where: { type: entities.inputs.UsersFilters } },
        resolve: async (source, args) => { /* custom logic */ },
      },
    },
  }),
  mutation: new GraphQLObjectType({
    name: 'Mutation',
    fields: entities.mutations,
  }),
  types: [...Object.values(entities.types), ...Object.values(entities.inputs)],
});
```

# Prisma Extension

Use Drizzle alongside Prisma, sharing the same database connection:

```typescript
import { PrismaClient } from '@prisma/client';
import { drizzle } from 'drizzle-orm/prisma/pg'; // or /mysql, /sqlite

const prisma = new PrismaClient().$extends(drizzle());
```

Add a Prisma generator to auto-generate Drizzle table definitions:

```prisma
generator drizzle {
  provider = "drizzle-prisma-generator"
  output   = "./drizzle"
}
```

Then use both APIs:

```typescript
// Prisma API
const user = await prisma.user.findFirst();

// Drizzle API (via extension)
const users = await prisma.$drizzle.select().from(User);
await prisma.$drizzle.insert().into(User).values({ name: 'John' });
```

**Limitations:** Relational queries not supported through the Prisma extension.

# ESLint Plugin

Prevent accidental bulk updates/deletes:

```bash
npm install eslint-plugin-drizzle
```

```yaml
# .eslintrc.yml
plugins:
  - drizzle
rules:
  drizzle/enforce-delete-with-where:
    - error
    - drizzleObjectName: db
  drizzle/enforce-update-with-where:
    - error
    - drizzleObjectName: db
```

<!--
Source references:
- https://orm.drizzle.team/docs/graphql
- https://orm.drizzle.team/docs/prisma
- https://orm.drizzle.team/docs/eslint-plugin
-->
