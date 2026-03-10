---
name: advanced-stories-videochats
description: Stories, video chats, and live streams (user-only)
---

# Stories & Video Chats

All methods in this reference are user-only.

## Stories

### Creating Stories

```ts
const story = await client.createStory(chatId,
  { photo: "/path/to/photo.jpg" }, // InputStoryContent
  {
    caption: "My story",
    parseMode: "HTML",
    privacy: { allowedUsers: [userId1, userId2] }, // StoryPrivacy
    activeFor: 86400,          // Duration in seconds
    highlight: true,           // Add to highlights
    isContentProtected: true,
    interactiveAreas: [],      // StoryInteractiveArea[]
  }
);
```

### Managing Stories

```ts
const story = await client.getStory(chatId, storyId);
const stories = await client.getStories(chatId, [id1, id2]);

await client.deleteStory(chatId, storyId);
await client.deleteStories(chatId, [id1, id2]);
```

### Highlights

```ts
await client.addStoryToHighlights(chatId, storyId);
await client.addStoriesToHighlights(chatId, [id1, id2]);
await client.removeStoryFromHighlights(chatId, storyId);
await client.removeStoriesFromHighlights(chatId, [id1, id2]);
```

### Story Updates

```ts
client.on("story", (ctx) => {
  const story = ctx.update.story;
});

client.on("deletedStory", (ctx) => {
  const { chatId, storyId } = ctx.update.deletedStory;
});
```

## Video Chats

### Starting & Scheduling

```ts
const vc = await client.startVideoChat(chatId, {
  title: "My Video Chat",
  isLiveStream: false,
});

const scheduled = await client.scheduleVideoChat(chatId,
  Math.floor(Date.now() / 1000) + 3600, // Start in 1 hour
  { title: "Upcoming Chat" }
);
```

### Joining & Leaving

```ts
const sdp = await client.joinVideoChat(vcId, sdpParams, {
  joinAs: "@channelId",
  inviteHash: "abc123",
  isAudioEnabled: true,
  isVideoEnabled: true,
});

await client.leaveVideoChat(vcId);
```

### Getting Info

```ts
const vc = await client.getVideoChat(vcId);
```

### Live Streams

```ts
await client.joinLiveStream(vcId);

const channels = await client.getLiveStreamChannels(vcId);

const segment = await client.downloadLiveStreamSegment(
  vcId, channelId, scale, timestamp,
  { quality: "high", signal: abortController.signal }
);
```

### Video Chat Updates

```ts
client.on("videoChat", (ctx) => {
  const vc = ctx.update.videoChat;
});
```

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/client/5_client.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/4_story_manager.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/3_video_chat_manager.ts
-->
