---
name: core-types
description: Key types - ID, FileSource, ParseMode, Reaction, ReplyMarkup
---

# Core Types

## ID

Used for identifying chats and users. Accepts:
- `number` - Numeric ID
- `string` - Username (with or without `@`)

```ts
type ID = number | string;

await client.sendMessage(123456789, "By ID");
await client.sendMessage("@username", "By username");
```

## FileSource

Flexible input for file operations:

```ts
type FileSource =
  | string                          // File ID, file path, or URL
  | URL                             // URL object
  | Uint8Array                      // Raw bytes
  | Iterable<Uint8Array>            // Sync chunk iterator
  | AsyncIterable<Uint8Array>       // Async chunk iterator
  | ReadableStream<Uint8Array>;     // Web Streams API

// Examples
await client.sendPhoto(chatId, "file_id_from_telegram");
await client.sendPhoto(chatId, "/path/to/photo.jpg");
await client.sendPhoto(chatId, new URL("https://example.com/photo.jpg"));
await client.sendPhoto(chatId, rawBytes);
```

## ParseMode

```ts
type ParseMode = "HTML" | "Markdown" | null;

// Set default in client constructor
const client = new Client({ parseMode: "HTML" });

// Override per message
await client.sendMessage(chatId, "<b>Bold</b>", { parseMode: "HTML" });
await client.sendMessage(chatId, "**Bold**", { parseMode: "Markdown" });
```

## Reaction

```ts
type Reaction =
  | { type: "emoji"; emoji: string }
  | { type: "customEmoji"; customEmojiId: string };

await client.addReaction(chatId, messageId, { type: "emoji", emoji: "👍" });
```

## ReplyMarkup

```ts
// Inline keyboard (bot-only)
const inlineKeyboard = {
  inlineKeyboard: [
    [
      { text: "Click me", callbackData: "action1" },
      { text: "Open URL", url: "https://example.com" },
    ],
    [
      { text: "Switch Inline", switchInlineQuery: "search" },
    ],
  ],
};

// Regular keyboard (bot-only)
const keyboard = {
  keyboard: [
    [{ text: "Button 1" }, { text: "Button 2" }],
    [{ text: "Share Contact", requestContact: true }],
    [{ text: "Share Location", requestLocation: true }],
  ],
  isResized: true,
  isOneTime: true,
  inputFieldPlaceholder: "Choose an option...",
};

// Remove keyboard
const removeKeyboard = { removeKeyboard: true };

// Force reply
const forceReply = { forceReply: true, inputFieldPlaceholder: "Type here..." };

await client.sendMessage(chatId, "Choose:", { replyMarkup: inlineKeyboard });
```

## InlineKeyboardButton Variants

| Type | Fields |
|------|--------|
| URL | `{ text, url }` |
| Callback | `{ text, callbackData }` |
| MiniApp | `{ text, miniApp }` |
| Login | `{ text, loginUrl }` |
| SwitchInline | `{ text, switchInlineQuery }` |
| SwitchInlineCurrentChat | `{ text, switchInlineQueryCurrentChat }` |
| Game | `{ text, callbackGame }` |
| Pay | `{ text, pay: true }` |

## Message Types

Messages are a discriminated union. Key content types:

| Type | Content Fields |
|------|----------------|
| `MessageText` | `text`, `entities` |
| `MessagePhoto` | `photo`, `caption`, `captionEntities` |
| `MessageVideo` | `video`, `caption`, `captionEntities` |
| `MessageDocument` | `document`, `caption`, `captionEntities` |
| `MessageSticker` | `sticker` |
| `MessageAnimation` | `animation`, `caption` |
| `MessageVoice` | `voice`, `caption` |
| `MessageAudio` | `audio`, `caption` |
| `MessageLocation` | `location` |
| `MessageContact` | `contact` |
| `MessagePoll` | `poll` |
| `MessageDice` | `dice` |
| `MessageVenue` | `venue` |
| `MessageInvoice` | `invoice` |

Common base fields on all messages: `id`, `chat`, `from`, `date`, `isOutgoing`, `replyToMessage`, `replyMarkup`, `views`, `forwards`, `reactions`.

## ChatAction

Typing indicators:

```ts
type ChatAction =
  | "type" | "uploadPhoto" | "recordVideo" | "uploadVideo"
  | "recordVoice" | "uploadAudio" | "uploadDocument"
  | "chooseSticker" | "findLocation" | "recordVideoNote"
  | "uploadVideoNote";

await client.sendChatAction(chatId, "type");
```

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/types/0_file_source.ts
- https://github.com/MTKruto/MTKruto/blob/main/types/0_parse_mode.ts
- https://github.com/MTKruto/MTKruto/blob/main/types/2_reply_markup.ts
- https://github.com/MTKruto/MTKruto/blob/main/types/6_message.ts
-->
