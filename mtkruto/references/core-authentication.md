---
name: core-authentication
description: Bot token and user phone authentication flows
---

# Authentication

## Bot Authentication

```ts
// Quick start (connect + auth in one call)
await client.start({ botToken: "123456:ABC-DEF" });

// Manual flow
await client.connect();
const result = await client.checkBotToken("123456:ABC-DEF");
// result: BotTokenCheckResult
```

## User Authentication

```ts
// Interactive start with callbacks
await client.start({
  phone: () => prompt("Phone number:"),
  code: () => prompt("Verification code:"),
  password: (hint) => prompt(`Password (hint: ${hint}):`),
});

// Or with static values
await client.start({
  phone: "+1234567890",
  code: "12345",
  password: "mypassword",
});
```

### Manual User Auth Flow

```ts
await client.connect();

// Step 1: Send verification code
await client.sendCode("+1234567890");

// Step 2: Check verification code
const codeResult = await client.checkCode("12345");
// codeResult: CodeCheckResult

// Step 3: If 2FA is enabled
const hint = await client.getPasswordHint();
const pwResult = await client.checkPassword("mypassword");
// pwResult: PasswordCheckResult
```

## Session Management

```ts
// Export session for persistence
const authString = await client.exportAuthString();
// Store authString securely

// Import session on next startup
await client.importAuthString(authString);
// Or pass during construction:
const client = new Client({ authString: "..." });

// Sign out (clears session)
await client.signOut();
```

## Get Current User

```ts
const me = await client.getMe();
// User { id, firstName, lastName, username, isBot, ... }
```

<!--
Source references:
- https://github.com/MTKruto/MTKruto/blob/main/client/5_client.ts
- https://github.com/MTKruto/MTKruto/blob/main/client/0_params.ts
-->
