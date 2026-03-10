---
name: features-messages
description: Sending, editing, deleting, forwarding, and managing messages
---

# Messages

## Sending Text Messages

```ts
const msg = await client.sendMessage(chatId, "Hello, world!");

// With options
const msg = await client.sendMessage(chatId, "<b>Bold</b> text", {
  parseMode: "HTML",
  disableNotification: true,
  isContentProtected: true,
  replyTo: { messageId: 42 },
  messageThreadId: 123,         // For topics/forums
  sendAs: "@channelUsername",   // Send on behalf of (user-only)
  effectId: 5107584321108051014, // Message effect
  sendAt: Math.floor(Date.now() / 1000) + 3600, // Schedule (user-only)
  replyMarkup: {
    inlineKeyboard: [[{ text: "Click", callbackData: "action" }]],
  },
  linkPreview: { url: "https://example.com", aboveText: true },
});
```

## Sending Media

```ts
// Photo
const photo = await client.sendPhoto(chatId, "/path/to/photo.jpg", {
  caption: "A photo",
  hasSpoiler: true,
  selfDestruct: { type: "timer", duration: 10 },
});

// Document
const doc = await client.sendDocument(chatId, fileSource, {
  caption: "Important file",
  fileName: "report.pdf",
  mimeType: "application/pdf",
  thumbnail: thumbnailSource,
});

// Video
const video = await client.sendVideo(chatId, videoSource, {
  caption: "Watch this",
  duration: 120,
  width: 1920,
  height: 1080,
  supportsStreaming: true,
  hasSpoiler: true,
});

// Sticker
await client.sendSticker(chatId, stickerFileId, { emoji: "😀" });

// Animation (GIF)
await client.sendAnimation(chatId, gifSource, { caption: "Funny GIF" });

// Voice
await client.sendVoice(chatId, voiceSource, { duration: 5 });

// Audio
await client.sendAudio(chatId, audioSource, {
  title: "Song Title",
  performer: "Artist",
  duration: 240,
});

// Video note (round video)
await client.sendVideoNote(chatId, videoNoteSource, {
  duration: 15,
  length: 240,
});

// Media group (album)
await client.sendMediaGroup(chatId, [
  { photo: photo1, caption: "First" },
  { photo: photo2 },
  { video: videoSource },
]);
```

## Sending Other Content

```ts
// Location
await client.sendLocation(chatId, 48.8566, 2.3522, {
  livePeriod: 3600,       // Live location duration in seconds
  heading: 90,            // Direction 1-350
  proximityAlertRadius: 100,
});

// Venue
await client.sendVenue(chatId, 48.8566, 2.3522, "Eiffel Tower", "Paris, France");

// Contact
await client.sendContact(chatId, "John", "+1234567890", {
  lastName: "Doe",
  vcard: "...",
});

// Dice
await client.sendDice(chatId, { emoji: "🎲" }); // 🎲 🎯 🏀 ⚽ 🎳 🎰

// Poll
await client.sendPoll(chatId, "Favorite color?", ["Red", "Blue", "Green"], {
  isAnonymous: false,
  type: "quiz",
  correctOptionIndex: 1,
  explanation: "Blue is best!",
  openPeriod: 300,
});

// Invoice (bot-only)
await client.sendInvoice(chatId, "Product", "Description", "payload", "USD",
  [{ label: "Item", amount: 1000 }],
  { providerToken: "..." }
);
```

## Editing Messages

```ts
// Edit text
await client.editMessageText(chatId, messageId, "Updated text", {
  parseMode: "HTML",
  replyMarkup: newKeyboard,
});

// Edit caption
await client.editMessageCaption(chatId, messageId, {
  caption: "New caption",
  parseMode: "HTML",
});

// Edit media
await client.editMessageMedia(chatId, messageId, { photo: newPhoto });

// Edit reply markup (buttons)
await client.editMessageReplyMarkup(chatId, messageId, {
  replyMarkup: newInlineKeyboard,
});

// Edit live location
await client.editMessageLiveLocation(chatId, messageId, newLat, newLng);

// Edit inline messages (bot-only)
await client.editInlineMessageText(inlineMessageId, "New text");
await client.editInlineMessageCaption(inlineMessageId, { caption: "New" });
await client.editInlineMessageMedia(inlineMessageId, { photo: newPhoto });
```

## Retrieving Messages

```ts
const msg = await client.getMessage(chatId, messageId);
const msgs = await client.getMessages(chatId, [id1, id2, id3]);
const msg = await client.resolveMessageLink("https://t.me/channel/123");

// Chat history (user-only)
const history = await client.getHistory(chatId, {
  limit: 50,
  offsetId: lastMessageId,
  offsetDate: timestamp,
});

// Search messages (user-only)
const results = await client.searchMessages({
  chatId: chatId,
  query: "search term",
  filter: "photo",
  limit: 20,
});
```

## Deleting Messages

```ts
await client.deleteMessage(chatId, messageId);
await client.deleteMessages(chatId, [id1, id2, id3]);
await client.deleteMessage(chatId, messageId, { onlyForMe: true });

// Delete all messages from a member (user-only)
await client.deleteChatMemberMessages(chatId, userId);
```

## Forwarding

```ts
await client.forwardMessage(fromChatId, toChatId, messageId, {
  disableNotification: true,
  dropSenderName: true,
  dropCaption: true,
});

await client.forwardMessages(fromChatId, toChatId, [id1, id2]);
```

## Pinning

```ts
await client.pinMessage(chatId, messageId, {
  bothSides: true,           // Private chats only
  disableNotification: true,
});
await client.unpinMessage(chatId, messageId);
await client.unpinMessages(chatId); // Unpin all
```

## Scheduled Messages

```ts
// Send a scheduled message early
await client.sendScheduledMessage(chatId, scheduledMessageId);
await client.sendScheduledMessages(chatId, [id1, id2]);

// Delete scheduled messages
await client.deleteScheduledMessage(chatId, scheduledMessageId);
```

## Typing Indicators

```ts
await client.sendChatAction(chatId, "type");
await client.sendChatAction(chatId, "uploadPhoto");
await client.sendChatAction(chatId, "recordVideo");
```

## Mark as Read (user-only)

```ts
await client.readMessages(chatId, untilMessageId);
```

## Upload Progress Tracking

```ts
const progressId = await client.getProgressId();
await client.sendPhoto(chatId, largeFile, { progressId });

// Listen for progress updates
client.on("uploadProgress", (ctx) => {
  const { uploaded, totalSize } = ctx.update;
  console.log(`${uploaded}/${totalSize} bytes`);
});
```

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/client/5_client.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/0_params.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/3_message_manager.ts
-->
