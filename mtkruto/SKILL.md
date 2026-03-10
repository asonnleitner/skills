---
name: mtkruto
description: Cross-runtime JavaScript library for building Telegram clients with MTProto
metadata:
  author: Andreas Sonnleitner
  version: "2026.3.10"
  source: Generated from https://github.com/MTKruto/MTKruto, scripts located at https://github.com/asonnleitner/skills
---

> The skill is based on MTKruto (pre-1.0), generated at 2026-03-10.

MTKruto is a cross-runtime JavaScript/TypeScript library for building Telegram clients. It supports Node.js, Deno, Bun, and browsers. It provides a high-level API on top of the Telegram MTProto protocol with a middleware-based update handling system (similar to grammY/Koa). Supports both bot and user authentication.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Client Setup | Initialization, configuration options, and connection | [core-client-setup](references/core-client-setup.md) |
| Authentication | Bot token and user phone auth flows, session management | [core-authentication](references/core-authentication.md) |
| Updates & Middleware | Handling updates, filters, commands, Composer, Context | [core-updates-middleware](references/core-updates-middleware.md) |
| Core Types | ID, FileSource, ParseMode, Reaction, ReplyMarkup, Message types | [core-types](references/core-types.md) |

## Features

### Messaging

| Topic | Description | Reference |
|-------|-------------|-----------|
| Messages | Send, edit, delete, forward, pin, schedule messages | [features-messages](references/features-messages.md) |
| Reactions & Polls | Message reactions, polls, voting | [features-reactions-polls](references/features-reactions-polls.md) |
| Inline & Callbacks | Inline queries, callback queries, bot commands | [features-inline-callback](references/features-inline-callback.md) |

### Chats & Files

| Topic | Description | Reference |
|-------|-------------|-----------|
| Chat Management | Create, configure, manage members, permissions, invite links | [features-chats](references/features-chats.md) |
| Forum Topics | Forum topic creation and management | [features-forums](references/features-forums.md) |
| File Operations | Upload, download, progress tracking | [features-files](references/features-files.md) |
| Storage | Storage providers for session persistence | [features-storage](references/features-storage.md) |

### Advanced

| Topic | Description | Reference |
|-------|-------------|-----------|
| Account Management | Profile, usernames, status, business features | [advanced-account](references/advanced-account.md) |
| Stories & Video Chats | Stories, video chats, live streams (user-only) | [advanced-stories-videochats](references/advanced-stories-videochats.md) |
| Errors & Raw API | Error handling, raw MTProto invocation, payments, gifts | [advanced-errors-raw-api](references/advanced-errors-raw-api.md) |
