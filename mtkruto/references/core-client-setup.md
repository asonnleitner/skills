---
name: core-client-setup
description: Client initialization, configuration, and connection management
---

# Client Setup & Configuration

MTKruto client setup for building Telegram bots and user clients.

## Installation

```ts
// Node.js / Bun
const { Client } = require("@mtkruto/node"); // npm install @mtkruto/node

// Deno
import { Client } from "https://deno.land/x/mtkruto/mod.ts";

// Browser
import { Client } from "https://esm.sh/jsr/@mtkruto/mtkruto";
```

## Client Construction

```ts
import { Client } from "@mtkruto/node";

const client = new Client({
  storage: new StorageMemory(),      // Storage provider (default: memory)
  apiId: 12345,                      // From my.telegram.org/apps
  apiHash: "abc123",                 // From my.telegram.org/apps
  parseMode: "HTML",                 // Default parse mode: "HTML" | "Markdown" | null
  appVersion: "1.0.0",              // App version string
  deviceModel: "Server",            // Device model
  defaultHandlers: true,            // Enable default error handlers
  outgoingMessages: false,          // Send outgoing messages as updates
  guaranteeUpdateDelivery: false,   // Order-sensitive update delivery
  dropPendingUpdates: true,         // Skip updates received while offline (default: true for bots)
  persistCache: false,              // Persist cache to storage instead of memory
  disableUpdates: false,            // Disable update processing
  authString: "...",                // Auto-import auth string on connect
  initialDc: "1",                   // Initial data center
});
```

## Connection

```ts
// Basic connect (no auth)
await client.connect();

// Connect + sign in (recommended for bots and user clients)
await client.start({ botToken: "BOT_TOKEN" });

// Disconnect
await client.disconnect();
```

## Key Properties

```ts
client.isConnected;    // boolean - connection state
client.isDisconnected; // boolean - disconnection state
```

## Key Options Explained

- **`persistCache: true`** - Recommended for user accounts, reduces memory usage, and helps when uptime is short.
- **`guaranteeUpdateDelivery: true`** - Ensures order-sensitive updates arrive before subsequent ones. Useful for desktop-style UI clients.
- **`outgoingMessages: true`** - Makes sent messages trigger update handlers. Bot business messages are excluded.
- **`dropPendingUpdates`** - Defaults to `true` for bots, `false` for users.

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/client/5_client.ts
- https://github.com/MTKruto/MTKruto/blob/main/README.md
-->
