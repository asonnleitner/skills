---
name: advanced-seeding
description: Deterministic database seeding with drizzle-seed
---

# Database Seeding

`drizzle-seed` generates realistic deterministic test data from your Drizzle schema.

## Setup

```bash
npm install drizzle-seed
```

## Basic Usage

```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import { seed } from 'drizzle-seed';
import * as schema from './schema';

const db = drizzle(process.env.DATABASE_URL);

async function main() {
  await seed(db, schema);
}

// Reset (clear) + seed
async function resetAndSeed() {
  await seed(db, schema, { count: 1000 }).refine((f) => ({
    users: {
      count: 500,
      columns: {
        name: f.fullName(),
        email: f.email(),
        age: f.int({ minValue: 18, maxValue: 90 }),
      },
    },
  }));
}
```

## Reset Database

Clear all data before seeding:

```typescript
import { reset } from 'drizzle-seed';

// PostgreSQL
await reset(db, schema);

// MySQL (tables order matters for foreign keys)
await reset(db, schema);

// SQLite
await reset(db, schema);
```

## Generator Functions

Key generators available in the `refine` callback:

| Generator | Description |
|-----------|-------------|
| `f.default()` | Default generator based on column type |
| `f.valuesFromArray({ values })` | Pick from array of values |
| `f.int({ minValue, maxValue })` | Integer range |
| `f.number({ minValue, maxValue, precision })` | Float range |
| `f.boolean()` | Random boolean |
| `f.string({ isUnique })` | Random string |
| `f.uuid()` | UUID v4 |
| `f.firstName()` | Realistic first name |
| `f.lastName()` | Realistic last name |
| `f.fullName()` | Full name |
| `f.email()` | Email address |
| `f.phoneNumber({ template })` | Phone number |
| `f.country()` | Country name |
| `f.city()` | City name |
| `f.streetAddress()` | Street address |
| `f.jobTitle()` | Job title |
| `f.loremIpsum({ sentencesCount })` | Lorem ipsum text |
| `f.date({ minDate, maxDate })` | Date |
| `f.timestamp()` | Timestamp |
| `f.json()` | JSON object |

## Weighted Random

```typescript
await seed(db, schema).refine((f) => ({
  users: {
    columns: {
      role: f.weightedRandom([
        { value: f.valuesFromArray({ values: ['user'] }), weight: 0.8 },
        { value: f.valuesFromArray({ values: ['admin'] }), weight: 0.2 },
      ]),
    },
  },
}));
```

## Deterministic Seeding

Data is deterministic by default — same schema produces same data. Override with a custom seed number:

```typescript
await seed(db, schema, { seed: 12345 });
```

<!--
Source references:
- https://orm.drizzle.team/docs/seed-overview
- https://orm.drizzle.team/docs/seed-functions
-->
