---
name: features-reactions-polls
description: Message reactions, polls, and voting
---

# Reactions & Polls

## Reactions

```ts
// Add a single reaction
await client.addReaction(chatId, messageId,
  { type: "emoji", emoji: "👍" },
  { isBig: true, addToRecents: true }
);

// Set all reactions at once (replaces existing)
await client.setReactions(chatId, messageId, [
  { type: "emoji", emoji: "👍" },
  { type: "emoji", emoji: "❤️" },
]);

// Remove a specific reaction
await client.removeReaction(chatId, messageId, { type: "emoji", emoji: "👍" });

// Get reactions on a message
const reactions = await client.getMessageReactions(chatId, messageId, {
  reaction: { type: "emoji", emoji: "👍" }, // Filter by type
  limit: 50,
  offset: "...",
});

// Set available reactions for a chat
await client.setAvailableReactions(chatId, "all");   // All reactions
await client.setAvailableReactions(chatId, "none");  // No reactions
await client.setAvailableReactions(chatId, [         // Specific reactions
  { type: "emoji", emoji: "👍" },
  { type: "emoji", emoji: "👎" },
]);
```

### Context Shortcuts

```ts
client.on("message", async (ctx) => {
  await ctx.react([{ type: "emoji", emoji: "👍" }]);
  await ctx.addReaction(ctx.msg.id, { type: "emoji", emoji: "❤️" });
  await ctx.removeReaction(ctx.msg.id, { type: "emoji", emoji: "❤️" });
});
```

### Listening for Reaction Updates

```ts
client.on("messageReactions", (ctx) => {
  const { chat, messageId, reactions } = ctx.update.messageReactions;
  // reactions: MessageReaction[] with user info
});

client.on("messageReactionCount", (ctx) => {
  const { chat, messageId, reactions } = ctx.update.messageReactionCount;
  // reactions: ReactionCount[] with counts only
});
```

## Polls

### Sending Polls

```ts
// Regular poll
await client.sendPoll(chatId, "What's your favorite?", ["Option A", "Option B"], {
  isAnonymous: true,
  allowMultipleAnswers: false,
  isClosed: false,
  openPeriod: 600,       // Auto-close after 10 minutes
});

// Quiz poll
await client.sendPoll(chatId, "What is 2+2?", ["3", "4", "5"], {
  type: "quiz",
  correctOptionIndex: 1,
  explanation: "Basic math: 2+2=4",
  explanationParseMode: "HTML",
  isAnonymous: false,
});
```

### Interacting with Polls (user-only)

```ts
await client.vote(chatId, messageId, [0]);         // Vote for first option
await client.vote(chatId, messageId, [0, 2]);       // Multi-select
await client.retractVote(chatId, messageId);         // Retract vote
```

### Stopping Polls

```ts
const finalPoll = await client.stopPoll(chatId, messageId);
```

### Poll Updates

```ts
client.on("poll", (ctx) => {
  const poll = ctx.update.poll;
  // Poll with updated results
});

client.on("pollAnswer", (ctx) => {
  const { from, optionIndexes } = ctx.update.pollAnswer;
  // Individual user's vote
});
```

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/client/5_client.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/2_reaction_manager.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/4_poll_manager.ts
-->
