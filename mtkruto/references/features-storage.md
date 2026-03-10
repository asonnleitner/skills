---
name: features-storage
description: Storage providers for session persistence
---

# Storage Providers

Storage is used for persisting authentication sessions and caches.

## Available Providers

### StorageMemory (default)

In-memory storage. Fast but volatile — sessions lost on restart.

```ts
import { Client, StorageMemory } from "@mtkruto/node";

const client = new Client({
  storage: new StorageMemory(),
});
```

### StorageLocalStorage (browser)

Uses the browser's `localStorage` API. Persists across page reloads.

```ts
import { Client, StorageLocalStorage } from "https://esm.sh/jsr/@mtkruto/mtkruto";

const client = new Client({
  storage: new StorageLocalStorage("my_bot"),
});
```

### StorageSessionStorage (browser)

Uses the browser's `sessionStorage` API. Cleared when session ends.

```ts
import { Client, StorageSessionStorage } from "https://esm.sh/jsr/@mtkruto/mtkruto";

const client = new Client({
  storage: new StorageSessionStorage("my_bot"),
});
```

### StorageIndexedDB (browser)

Uses IndexedDB. Best browser option for large data and file support.

```ts
import { Client, StorageIndexedDB } from "https://esm.sh/jsr/@mtkruto/mtkruto";

const client = new Client({
  storage: new StorageIndexedDB("my_bot"),
});
```

## Auth String Persistence

For server environments without persistent storage, export/import auth strings:

```ts
// After first auth, export the session
const authString = await client.exportAuthString();
// Store in database, file, env var, etc.

// On next startup, import
const client = new Client({ authString: storedAuthString });
await client.connect();

// Or import manually
await client.importAuthString(storedAuthString);
```

## Storage Interface

Custom storage providers must implement:

```ts
interface Storage {
  initialize(): MaybePromise<void>;
  set(key: readonly StorageKeyPart[], value: unknown): MaybePromise<void>;
  incr(key: readonly StorageKeyPart[], by: number): MaybePromise<void>;
  get<T>(key: readonly StorageKeyPart[]): MaybePromise<T | null>;
  getMany<T>(prefix: GetManyFilter, params?: { limit?: number; reverse?: boolean }):
    MaybePromise<Generator<[readonly StorageKeyPart[], T]> | AsyncGenerator<...>>;
  branch(id: string): Storage;
  supportsFiles: boolean;
  mustSerialize: boolean;
  isMemory: boolean;
}
```

- Uses composite key arrays (`StorageKeyPart = string | number | bigint`)
- `branch(id)` creates isolated namespaces
- `mustSerialize` indicates if values need JSON serialization
- `supportsFiles` indicates file blob storage support

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/storage/0_storage.ts
- https://github.com/MTKruto/MTKruto/blob/main/storage/2_storage_memory.ts
- https://github.com/MTKruto/MTKruto/blob/main/storage/2_storage_indexed_db.ts
- https://github.com/MTKruto/MTKruto/blob/main/storage/2_storage_local_storage.ts
-->
