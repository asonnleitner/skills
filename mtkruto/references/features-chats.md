---
name: features-chats
description: Chat creation, management, members, permissions, and invite links
---

# Chat Management

## Get Chat Info

```ts
const chat = await client.getChat(chatId);
// Chat: ChatChannel | ChatSupergroup | ChatGroup | ChatPrivate

const me = await client.getMe();
```

## Chat List (user-only)

```ts
const chats = await client.getChats({ from: "main", limit: 50 });
const archivedChats = await client.getChats({ from: "archived" });
const pinnedChats = await client.getPinnedChats("main");
const settings = await client.getChatSettings(chatId);
const commonChats = await client.getCommonChats(userId, { limit: 10 });
const inactiveChats = await client.getInactiveChats();
```

## Creating Chats (user-only)

```ts
// Group
const group = await client.createGroup("Group Name", {
  users: [userId1, userId2],
  messageTtl: 86400,
});

// Supergroup
const supergroup = await client.createSupergroup("Super Name", {
  description: "A supergroup",
  isForum: true,
  messageTtl: 604800,
});

// Channel
const channel = await client.createChannel("Channel Name", {
  description: "A channel",
});
```

## Chat Settings

```ts
await client.setChatTitle(chatId, "New Title");
await client.setChatDescription(chatId, "New description");
await client.setChatPhoto(chatId, photoSource);
await client.deleteChatPhoto(chatId);
await client.setMessageTtl(chatId, 86400); // TTL in seconds (user-only)
await client.setChatStickerSet(chatId, "sticker_set_name");
await client.deleteChatStickerSet(chatId);
await client.setAvailableReactions(chatId, "all"); // "all" | "none" | Reaction[]
```

## Group/Supergroup Features (user-only)

```ts
// Slow mode
await client.setSlowMode(chatId, 30); // SlowModeDuration: 10|30|60|300|900|3600
await client.disableSlowMode(chatId);

// Forum topics
await client.enableTopics(chatId, true); // isShownAsTabs
await client.disableTopics(chatId);

// Anti-spam
await client.enableAntispam(chatId);
await client.disableAntispam(chatId);

// Member list visibility
await client.hideMemberList(chatId);
await client.showMemberList(chatId);

// Sharing
await client.enableSharing(chatId);
await client.disableSharing(chatId);

// Business bots
await client.enableBusinessBots(chatId);
await client.disableBusinessBots(chatId);
```

## Channel Features (user-only)

```ts
await client.enableSignatures(chatId, { showAuthorProfile: true });
await client.disableSignatures(chatId);
await client.setDiscussionChat(channelId, groupId);
const suggestions = await client.getDiscussionChatSuggestions();
```

## Chat Lifecycle

```ts
await client.openChat(chatId);          // Open chat for real-time updates
await client.closeChat(chatId);         // Close chat
await client.joinChat(chatId);          // Join (user-only)
await client.leaveChat(chatId);         // Leave
await client.deleteChat(chatId);        // Delete (user-only)
await client.archiveChat(chatId);       // Archive (user-only)
await client.unarchiveChat(chatId);     // Unarchive (user-only)
await client.transferChatOwnership(chatId, userId, password); // Transfer (user-only)
```

## Members

```ts
const member = await client.getChatMember(chatId, userId);
const members = await client.getChatMembers(chatId, { offset: 0, limit: 50 });
const admins = await client.getChatAdministrators(chatId);
```

### ChatMember Types

```ts
// member.status:
// "creator"       - Chat owner
// "administrator" - Admin with rights
// "member"        - Regular member (may have until, tag)
// "restricted"    - Restricted member with limited rights
// "left"          - Left the chat
// "banned"        - Banned from the chat
```

### Adding/Removing Members

```ts
await client.addChatMember(chatId, userId, { historyLimit: 100 });
await client.addChatMembers(chatId, [userId1, userId2]);
await client.banChatMember(chatId, memberId, {
  until: Math.floor(Date.now() / 1000) + 86400, // 24h ban
  deleteMessages: true,
});
await client.unbanChatMember(chatId, memberId);
await client.kickChatMember(chatId, memberId); // Ban + immediate unban
```

### Permissions & Promotion

```ts
// Restrict a member
await client.setChatMemberRights(chatId, memberId, {
  rights: {
    canSendMessages: true,
    canSendMedia: false,
    canSendPolls: false,
    canSendStickers: false,
    canSendAnimations: false,
    canAddLinkPreviews: false,
    canChangeInfo: false,
    canInviteUsers: false,
    canPinMessages: false,
    canManageTopics: false,
  },
  until: Math.floor(Date.now() / 1000) + 3600,
});

// Promote to admin
await client.promoteChatMember(chatId, userId, {
  isAnonymous: false,
  canManageChat: true,
  canDeleteMessages: true,
  canManageVideoChats: true,
  canRestrictMembers: true,
  canPromoteMembers: false,
  canChangeInfo: true,
  canInviteUsers: true,
  canPostMessages: true,   // Channels only
  canEditMessages: true,   // Channels only
  canPinMessages: true,    // Groups/supergroups only
  canManageTopics: true,   // Supergroups only
  canPostStories: true,
  canEditStories: true,
  canDeleteStories: true,
  title: "Moderator",
});

// Set member tag
await client.setChatMemberTag(chatId, userId, { tag: "VIP" });
```

## Invite Links

```ts
const link = await client.createInviteLink(chatId, {
  title: "Invite Link",
  expireAt: Math.floor(Date.now() / 1000) + 86400,
  limit: 100,
  isApprovalRequired: false,
});

const links = await client.getCreatedInviteLinks(chatId, {
  limit: 50,
  isRevoked: false,
});
```

## Join Requests (user-only)

```ts
const requests = await client.getJoinRequests(chatId, {
  inviteLink: "https://t.me/+abc123",
  search: "John",
  limit: 50,
});

await client.approveJoinRequest(chatId, userId);
await client.declineJoinRequest(chatId, userId);
await client.approveJoinRequests(chatId); // Approve all
await client.declineJoinRequests(chatId); // Decline all

await client.enableJoinRequests(chatId);
await client.disableJoinRequests(chatId);
```

## User Management

```ts
// Contacts (user-only)
const contacts = await client.getContacts();
await client.addContact(userId, { firstName: "John", lastName: "Doe" });
await client.deleteContact(userId);

// Blocking (user-only)
await client.blockUser(userId);
await client.unblockUser(userId);
```

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/client/5_client.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/4_chat_manager.ts
- https://github.com/MTKruto/MTKruto/blob/main/types/2_chat_member.ts
-->
