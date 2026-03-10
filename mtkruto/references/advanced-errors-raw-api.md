---
name: advanced-errors-raw-api
description: Error handling, raw API invocation, and payments
---

# Errors, Raw API & Payments

## Error Types

```ts
import { errors } from "@mtkruto/node";

// FloodWait - rate limited
try {
  await client.sendMessage(chatId, "Hello");
} catch (e) {
  if (e instanceof errors.FloodWait) {
    console.log(`Wait ${e.seconds} seconds`);
    await new Promise(r => setTimeout(r, e.seconds * 1000));
  }
}

// Migration errors - need to switch DC
// Migrate, UserMigrate, PhoneMigrate, FileMigrate, StatsMigrate
if (e instanceof errors.Migrate) {
  console.log(`Migrate to DC ${e.dc}`);
}

// AuthKeyUnregistered - session expired
// SessionRevoked - session was revoked
```

## Invoke Error Handlers

```ts
// Add middleware for invoke errors
client.invoke.use(async (ctx, next) => {
  try {
    return await next();
  } catch (e) {
    if (e instanceof errors.FloodWait) {
      // Auto-retry after wait
      await new Promise(r => setTimeout(r, e.seconds * 1000));
      return await next();
    }
    throw e;
  }
});
```

## Raw API Invocation

Call any Telegram MTProto API method directly:

```ts
import { Api } from "@mtkruto/node";

// Ping
const pong = await client.invoke({
  _: "ping",
  ping_id: getRandomId(),
});

// Any TL function
const result = await client.invoke({
  _: "messages.getDialogs",
  offset_date: 0,
  offset_id: 0,
  offset_peer: { _: "inputPeerEmpty" },
  limit: 100,
  hash: 0n,
});

// With DC specification
const result = await client.invoke(func, { dc: "2" });

// Get InputPeer for raw API
const peer = await client.getInputPeer(chatId);
const channel = await client.getInputChannel(chatId);
const user = await client.getInputUser(userId);
```

## Payments

### Sending Invoices (bot-only)

```ts
await client.sendInvoice(chatId, "Product", "Description", "payload", "USD",
  [{ label: "Item", amount: 1000 }], // $10.00
  {
    providerToken: "...",
    maxTipAmount: 500,
    suggestedTipAmounts: [100, 200, 500],
    photoUrl: "https://example.com/product.jpg",
    needName: true,
    needEmail: true,
    needShippingAddress: true,
    isFlexible: true,
  }
);
```

### Pre-checkout Query (bot-only)

```ts
client.on("preCheckoutQuery", async (ctx) => {
  // Validate the order
  const query = ctx.update.preCheckoutQuery;

  // Approve
  await ctx.answerPreCheckoutQuery(true);

  // Or decline with error
  await ctx.answerPreCheckoutQuery(false, {
    error: "Item out of stock",
  });
});
```

### Refunds (bot-only)

```ts
await client.refundStarPayment(userId, telegramPaymentChargeId);
```

## Gifts

```ts
const gifts = await client.getGifts();

await client.sendGift(chatId, giftId, {
  message: "Happy birthday!",
  parseMode: "HTML",
  isPrivate: true,
  upgrade: true,
});

// user-only
const claimed = await client.getClaimedGifts(chatId, { limit: 50 });
await client.sellGift(inputGift);
await client.craftGifts([gift1, gift2]);
```

## Other Utilities

```ts
// Network statistics
const stats = await client.getNetworkStatistics();

// Voice transcription (user-only)
const transcription = await client.transcribeVoice(chatId, messageId);

// Mini apps (user-only)
const miniApp = await client.openMiniApp(botId, chatId, {
  mode: "default",
  url: "https://example.com/app",
  startParameter: "start",
});

// Link preview (user-only)
const preview = await client.getLinkPreview("Check https://example.com", {
  parseMode: "HTML",
});

// Start a bot (user-only)
await client.startBot(botId, { deeplink: "start_param" });

// Translations (user-only)
const translations = await client.getTranslations({
  platform: "android",
  language: "en",
});
```

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/client/5_client.ts
- https://github.com/MTKruto/MTKruto/blob/main/4_errors.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/2_payment_manager.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/4_gift_manager.ts
-->
