# Migrating from near-api-js

If you are coming from `near-api-js`, here is how common tasks translate to `near-kit`.

### Initialization

**near-api-js:**

```typescript
const { connect, keyStores, KeyPair } = require("near-api-js");
const keyStore = new keyStores.InMemoryKeyStore();
await keyStore.setKey("testnet", "alice.testnet", KeyPair.fromString("..."));
const near = await connect({ ... });
const account = await near.account("alice.testnet");
```

**near-kit:**

```typescript
const near = new Near({
  network: "testnet",
  privateKey: "ed25519:...",
})
```

### Calling a Contract

**near-api-js:**

```typescript
await account.functionCall({
  contractId: "market.near",
  methodName: "buy",
  args: { id: "1" },
  gas: "50000000000000", // 50 Tgas
  attachedDeposit: utils.format.parseNearAmount("1"),
})
```

**near-kit:**

```typescript
await near
  .transaction("alice.testnet")
  .functionCall(
    "market.near",
    "buy",
    { id: "1" },
    { gas: "50 Tgas", attachedDeposit: "1 NEAR" }
  )
  .send()
```

### Sending Money

**near-api-js:**

```typescript
await account.sendMoney("bob.near", utils.format.parseNearAmount("5"))
```

**near-kit:**

```typescript
await near.send("bob.near", "5 NEAR")
```
