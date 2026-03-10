---
name: core-updates-middleware
description: Handling updates with middleware, filters, commands, and the Context object
---

# Updates & Middleware

MTKruto uses a middleware pattern (like Koa/grammY) for handling Telegram updates.

## Basic Update Handling

```ts
// Handle any update
client.use((ctx, next) => {
  console.log("Update received:", ctx.update);
  return next(); // Call next to pass to subsequent handlers
});
```

## Filter Updates by Type

```ts
// Level 1 filters - match update type
client.on("message", (ctx) => { /* new message */ });
client.on("editedMessage", (ctx) => { /* edited message */ });
client.on("callbackQuery", (ctx) => { /* inline button pressed */ });
client.on("inlineQuery", (ctx) => { /* inline query received */ });
client.on("chosenInlineResult", (ctx) => { /* inline result chosen */ });
client.on("chatMember", (ctx) => { /* member status changed */ });
client.on("myChatMember", (ctx) => { /* bot's member status changed */ });
client.on("joinRequest", (ctx) => { /* join request */ });
client.on("messageReactions", (ctx) => { /* reactions updated */ });
client.on("story", (ctx) => { /* new story */ });
client.on("connectionState", (ctx) => { /* connection state changed */ });

// Level 2 filters - match specific content type
client.on("message:text", (ctx) => { /* text message */ });
client.on("message:photo", (ctx) => { /* photo message */ });
client.on("message:video", (ctx) => { /* video message */ });
client.on("message:document", (ctx) => { /* document message */ });
client.on("message:sticker", (ctx) => { /* sticker message */ });
client.on("message:animation", (ctx) => { /* GIF/animation */ });
client.on("message:voice", (ctx) => { /* voice message */ });
client.on("message:audio", (ctx) => { /* audio file */ });
client.on("message:location", (ctx) => { /* location */ });
client.on("message:contact", (ctx) => { /* contact */ });
client.on("message:poll", (ctx) => { /* poll */ });
client.on("message:dice", (ctx) => { /* dice */ });

// Shorthand: matches message, editedMessage, and scheduledMessage
client.on(":text", (ctx) => { /* text in any message variant */ });
client.on(":photo", (ctx) => { /* photo in any message variant */ });

// Callback query sub-filters
client.on("callbackQuery:data", (ctx) => { /* callback with data */ });
client.on("callbackQuery:message", (ctx) => { /* callback with message */ });
```

## Command Handling

```ts
// Match /start command
client.command("start", (ctx) => {
  await ctx.reply("Welcome!");
});

// Multiple commands
client.command(["help", "info"], (ctx) => {
  await ctx.reply("Help message");
});

// Regex command matching
client.command(/^admin_/, (ctx) => {
  // matches /admin_ban, /admin_kick, etc.
});

// Custom command prefix (default: "/" for bots, "\\" for users)
client.command({ names: "start", prefixes: ["!", "/"] }, (ctx) => {
  // matches both !start and /start
});

// Set command prefix for all commands on a composer
const composer = new Composer();
composer.prefixes = "!";
```

## Callback Query Handling

```ts
// Match specific callback data
client.callbackQuery("approve", async (ctx) => {
  await ctx.answerCallbackQuery({ text: "Approved!" });
});

// Regex matching
client.callbackQuery(/^action:(.+)$/, async (ctx) => {
  await ctx.answerCallbackQuery();
});

// Multiple patterns
client.callbackQuery(["yes", "no"], async (ctx) => {
  await ctx.answerCallbackQuery();
});
```

## Inline Query Handling

```ts
client.inlineQuery("search", async (ctx) => {
  await ctx.answerInlineQuery(results);
});

client.inlineQuery(/.*/, async (ctx) => {
  // Match any inline query
});
```

## Context Object

The `Context` object wraps updates with shortcuts:

```ts
client.on("message", (ctx) => {
  ctx.update;         // The raw Update object
  ctx.msg;            // Shortcut: message from message/editedMessage/callbackQuery
  ctx.message;        // The message (only for "message" updates)
  ctx.editedMessage;  // The edited message
  ctx.callbackQuery;  // The callback query
  ctx.inlineQuery;    // The inline query
  ctx.chat;           // The chat where the update occurred
  ctx.from;           // The user who triggered the update
  ctx.me;             // The current bot/user
});
```

### Context Reply Methods

```ts
client.on("message", async (ctx) => {
  // Reply methods auto-fill chatId and replyTo
  await ctx.reply("Hello!");
  await ctx.replyPhoto(photoSource);
  await ctx.replyDocument(fileSource);
  await ctx.replyVideo(videoSource);
  await ctx.replySticker(stickerSource);
  await ctx.replyLocation(lat, lng);
  await ctx.replyPoll("Question?", ["Yes", "No"]);
  await ctx.replyDice();
  await ctx.replyVoice(voiceSource);
  await ctx.replyAudio(audioSource);
  await ctx.replyAnimation(gifSource);
  await ctx.replyVideoNote(videoNoteSource);
  await ctx.replyContact("John", "+1234567890");
  await ctx.replyVenue(lat, lng, "Place", "Address");
  await ctx.replyMediaGroup([media1, media2]);
  await ctx.replyInvoice(title, desc, payload, currency, prices);

  // Control quoting behavior
  await ctx.reply("No quote", { isQuoted: false });

  // Other context shortcuts
  await ctx.delete();           // Delete the triggering message
  await ctx.forward(targetId);  // Forward the triggering message
  await ctx.pin();              // Pin the triggering message
  await ctx.unpin();            // Unpin the triggering message
  await ctx.react([{ type: "emoji", emoji: "👍" }]);
  await ctx.read();             // Mark as read (user-only)
  await ctx.banSender();        // Ban the sender
  await ctx.kickSender();       // Kick the sender
});
```

## Composer Pattern

```ts
import { Composer } from "@mtkruto/node";

// Create a separate composer for modular handlers
const adminComposer = new Composer();
adminComposer.command("ban", banHandler);
adminComposer.command("kick", kickHandler);

// Attach to main client
client.use(adminComposer);
```

## Custom Filters

```ts
client.filter(
  (ctx) => ctx.from?.isPremium === true,
  (ctx) => {
    // Only premium users reach here
  }
);

// Branch: different handlers based on predicate
client.branch(
  (ctx) => ctx.from?.isBot === true,
  (ctx) => { /* is a bot */ },
  (ctx) => { /* is a user */ }
);
```

## Update Types

All possible update types:

| Update | Description |
|--------|-------------|
| `connectionState` | Client connection state changed |
| `authorizationState` | Auth state changed |
| `message` | New message |
| `editedMessage` | Message was edited |
| `scheduledMessage` | Message was scheduled |
| `deletedMessages` | Messages were deleted |
| `callbackQuery` | Inline button pressed |
| `inlineQuery` | Inline query received |
| `chosenInlineResult` | Inline result chosen |
| `chatMember` | Chat member status changed |
| `myChatMember` | Bot's own status changed |
| `joinRequest` | Join request received |
| `messageReactions` | Reactions on a message changed |
| `messageReactionCount` | Reaction counts changed |
| `story` / `deletedStory` | Story events |
| `videoChat` | Video chat events |
| `poll` / `pollAnswer` | Poll events |
| `businessConnection` | Business connection events |
| `preCheckoutQuery` | Pre-checkout query |
| `uploadProgress` | File upload progress |

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/client/4_composer.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/2_context.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/3_filters.ts
- https://github.com/MTKruto/MTKruto/blob/main/types/8_update.ts
-->
