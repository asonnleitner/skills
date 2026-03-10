---
name: features-inline-callback
description: Inline queries, callback queries, and bot commands
---

# Inline Queries, Callbacks & Bot Commands

## Callback Queries

When a user clicks an inline keyboard button with `callbackData`:

```ts
// Answer with text
client.callbackQuery("approve", async (ctx) => {
  await ctx.answerCallbackQuery({ text: "Approved!" });
});

// Answer with alert popup
client.callbackQuery("info", async (ctx) => {
  await ctx.answerCallbackQuery({
    text: "This is important info",
    isAlert: true,
  });
});

// Answer with URL
client.callbackQuery("open", async (ctx) => {
  await ctx.answerCallbackQuery({ url: "https://example.com" });
});

// Edit the message that had the button
client.callbackQuery("toggle", async (ctx) => {
  await ctx.answerCallbackQuery();
  await ctx.editMessageText(ctx.msg.id, "Updated text", {
    replyMarkup: newKeyboard,
  });
});

// Direct method call (bot-only)
await client.answerCallbackQuery(callbackQueryId, {
  text: "Done!",
  cacheTime: 60,
});
```

## Sending Callback Queries (user-only)

```ts
const answer = await client.sendCallbackQuery(botId, messageId, {
  type: "data",
  data: "button_data",
});
// answer: CallbackQueryAnswer { text?, isAlert?, url? }
```

## Inline Queries

### Answering Inline Queries (bot-only)

```ts
client.inlineQuery(/.*/, async (ctx) => {
  const query = ctx.update.inlineQuery.query;

  await ctx.answerInlineQuery([
    // Article result
    {
      type: "article",
      id: "1",
      title: "Result Title",
      description: "Result description",
      messageContent: { type: "text", text: "Sent text" },
    },
    // Photo result
    {
      type: "photo",
      id: "2",
      photoUrl: "https://example.com/photo.jpg",
      thumbnailUrl: "https://example.com/thumb.jpg",
    },
  ], {
    cacheTime: 300,
    isPersonal: true,
    nextOffset: "page2",
    button: {
      text: "Open in bot",
      startParameter: "inline",
    },
  });
});

// Direct method call
await client.answerInlineQuery(inlineQueryId, results, params);
```

### Sending Inline Queries (user-only)

```ts
const answer = await client.sendInlineQuery(botId, chatId, {
  query: "search term",
  offset: "",
});
// answer: InlineQueryAnswer { results, nextOffset, ... }
```

### Editing Inline Messages (bot-only)

```ts
// After chosen inline result
client.on("chosenInlineResult", async (ctx) => {
  const inlineMessageId = ctx.update.chosenInlineResult.inlineMessageId;

  // Edit via context
  await ctx.editInlineMessageText("Updated text");
  await ctx.editInlineMessageCaption({ caption: "New caption" });
  await ctx.editInlineMessageMedia({ photo: newPhoto });
  await ctx.editInlineMessageReplyMarkup({ replyMarkup: newKeyboard });

  // Or via client
  await client.editInlineMessageText(inlineMessageId, "Updated");
  await client.editInlineMessageLiveLocation(inlineMessageId, lat, lng);
});
```

## Bot Commands

### Set Commands (bot-only)

```ts
await client.setMyCommands([
  { command: "start", description: "Start the bot" },
  { command: "help", description: "Show help" },
  { command: "settings", description: "Bot settings" },
], {
  languageCode: "en",
  scope: { type: "default" },
});

// Scoped commands
await client.setMyCommands(adminCommands, {
  scope: { type: "chatAdministrators" },
});
await client.setMyCommands(groupCommands, {
  scope: { type: "allGroupChats" },
});
```

### Get Commands

```ts
const commands = await client.getMyCommands({ languageCode: "en" });
```

### Bot Info (bot-only)

```ts
// Name
await client.setMyName({ name: "My Bot", languageCode: "en" });
const name = await client.getMyName();

// Description (shown on bot profile)
await client.setMyDescription({ description: "A helpful bot" });
const desc = await client.getMyDescription();

// Short description (shown in inline placeholder)
await client.setMyShortDescription({ shortDescription: "Quick helper" });
const shortDesc = await client.getMyShortDescription();
```

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/client/5_client.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/4_callback_query_manager.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/4_inline_query_manager.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/2_bot_info_manager.ts
-->
