---
name: advanced-account
description: Account profile, username, status, and business features
---

# Account Management

## Profile Updates

```ts
await client.updateProfile({
  firstName: "John",
  lastName: "Doe",
  bio: "Hello, I'm John!",
});
```

## Username Management

```ts
await client.setUsername("my_username");
await client.removeUsername();
const available = await client.checkUsername("desired_name");
await client.checkUsername("channel_name", { chatId: channelId }); // Check for chat

// Collectible usernames
await client.showUsername(chatId, "username");
await client.hideUsername(chatId, "username");
await client.reorderUsernames(chatId, ["first", "second"]);
await client.hideUsernames(chatId);
```

## Personal Profile (user-only)

```ts
await client.setBirthday({ birthday: { day: 1, month: 1, year: 2000 } });
await client.setBirthday({}); // Remove birthday

await client.setPersonalChannel({ chatId: channelId });
await client.setPersonalChannel({}); // Remove
```

## Name & Profile Colors

```ts
await client.setNameColor(1, { customEmojiId: "emoji_id" });
await client.setProfileColor(2, { customEmojiId: "emoji_id" });
```

## Online Status

```ts
await client.setOnline(true);
await client.setOnline(false);
```

## Emoji Status

```ts
await client.setEmojiStatus("emoji_id", {
  until: Math.floor(Date.now() / 1000) + 3600,
});

// Bot-only: set for another user
await client.setUserEmojiStatus(userId, "emoji_id");
```

## Business Features

```ts
// Business location
await client.setLocation({
  address: "123 Main St",
  latitude: 40.7128,
  longitude: -74.0060,
});
await client.setLocation({}); // Remove

// Business working hours
await client.setWorkingHours({
  workingHours: {
    timezone: "America/New_York",
    intervals: [{ startMinute: 540, endMinute: 1020 }],
  },
});

// Business connections (bot-only)
const conn = await client.getBusinessConnection(connectionId);
await client.pauseBusinessBotConnection(chatId);
await client.resumeBusinessBotConnection(chatId);
```

## Sponsored Messages (user-only)

```ts
await client.enableSponsoredMessages();
await client.disableSponsoredMessages();
```

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/client/5_client.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/2_account_manager.ts
-->
