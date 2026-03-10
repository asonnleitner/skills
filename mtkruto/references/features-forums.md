---
name: features-forums
description: Forum topic creation and management
---

# Forum Topics

Forum topics are available in supergroups with topics enabled.

## Enable/Disable Topics (user-only)

```ts
await client.enableTopics(chatId, true);  // isShownAsTabs
await client.disableTopics(chatId);
```

## Create & Edit Topics

```ts
const topic = await client.createTopic(chatId, "Topic Title", {
  color: 0x6FB9F0,          // Icon color
  customEmojiId: "emoji_id", // Custom emoji icon
  sendAs: "@channelId",      // Create on behalf of (user-only)
});
// topic: { id, title, iconColor, iconCustomEmojiId, ... }

const updated = await client.editTopic(chatId, topicId, "New Title", {
  customEmojiId: "new_emoji_id",
});
```

## Topic Management

```ts
// Close/reopen
await client.closeTopic(chatId, topicId);
await client.reopenTopic(chatId, topicId);

// Pin/unpin
await client.pinTopic(chatId, topicId);
await client.unpinTopic(chatId, topicId);

// General topic
await client.hideGeneralTopic(chatId);
await client.showGeneralTopic(chatId);
```

## Sending Messages to Topics

```ts
await client.sendMessage(chatId, "Hello topic!", {
  messageThreadId: topicId,
});
```

## Context Shortcuts

```ts
client.on("message", async (ctx) => {
  // ctx.msg.threadId gives the topic ID for topic messages
  await ctx.createTopic("New Topic");
  await ctx.editTopic(topicId, "Renamed");
  await ctx.closeTopic(topicId);
  await ctx.reopenTopic(topicId);
  await ctx.pinTopic(topicId);
  await ctx.unpinTopic(topicId);
  await ctx.hideGeneralTopic();
  await ctx.showGeneralTopic();
});
```

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/client/5_client.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/4_forum_manager.ts
-->
