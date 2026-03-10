---
name: features-files
description: File upload, download, and progress tracking
---

# File Operations

## Downloading Files

Files are downloaded by file ID (obtained from received messages).

```ts
// Download as async generator (streaming)
for await (const chunk of client.download(fileId)) {
  // chunk: Uint8Array
  process.stdout.write(chunk);
}

// Download with options
for await (const chunk of client.download(fileId, {
  chunkSize: 512 * 1024,  // 512 KB chunks
  offset: 0,
  signal: abortController.signal,
})) {
  // process chunk
}

// Collect all chunks
const chunks: Uint8Array[] = [];
for await (const chunk of client.download(fileId)) {
  chunks.push(chunk);
}
const fullFile = Buffer.concat(chunks);

// Download a single chunk
const chunk = await client.downloadChunk(fileId, {
  chunkSize: 1024 * 1024,
  offset: 0,
});
```

## File Sources for Uploading

Any `FileSource` can be used when sending media:

```ts
// File ID (re-send existing Telegram file)
await client.sendPhoto(chatId, "AgACAgIAAxk...");

// File path
await client.sendPhoto(chatId, "/path/to/photo.jpg");

// URL
await client.sendPhoto(chatId, "https://example.com/image.png");

// Raw bytes
await client.sendPhoto(chatId, new Uint8Array([...]));

// ReadableStream
const stream = fs.createReadStream("large-file.zip");
await client.sendDocument(chatId, stream);

// Async iterator
async function* generateChunks() {
  yield chunk1;
  yield chunk2;
}
await client.sendDocument(chatId, generateChunks());
```

## Upload Options

All send methods support upload configuration:

```ts
await client.sendDocument(chatId, fileSource, {
  fileName: "report.pdf",
  fileSize: 1024000,           // Helps with progress calculation
  mimeType: "application/pdf",
  chunkSize: 256 * 1024,       // Upload chunk size
  signal: abortController.signal,
  thumbnail: thumbnailSource,  // Thumbnail (not URL)
});
```

## Upload Progress Tracking

```ts
// Get a progress ID
const progressId = await client.getProgressId();

// Send with progress tracking
await client.sendDocument(chatId, largeFile, { progressId });

// Listen for progress updates
client.on("uploadProgress", (ctx) => {
  const progress = ctx.update;
  // progress: { uploaded: number, totalSize: number }
});
```

## Stickers

```ts
// Get sticker set
const set = await client.getStickerSet("sticker_set_name");

// Get custom emoji stickers
const stickers = await client.getCustomEmojiStickers("emoji_id");
const stickers = await client.getCustomEmojiStickers(["id1", "id2"]);
```

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/client/5_client.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/2_file_manager.ts
- https://github.com/MTKruto/MTKruto/blob/main/types/0_file_source.ts
-->
