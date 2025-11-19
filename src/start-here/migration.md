# Migrating from near-api-js

If you are coming from `near-api-js`, you will find `near-kit` to be more concise and less prone to "unit arithmetic" errors.

## Core Philosophy Shifts

1.  **No `Account` Object:** In `near-api-js`, you create an `Account` object and call methods on it. In `near-kit`, you use the central `Near` instance and pass the signer's ID as an argument.

2.  **Strings, not BigInts:** You never need to import `utils.format` to calculate gas or storage costs. Just use type-safe strings.

3.  **Fluent Builder:** Instead of passing massive configuration objects, you chain readable methods.

---

## Side-by-Side Comparisons

### 1. Connecting & Keys

**near-api-js:**
You have to manually assemble the Account, JsonRpcProvider, and KeyPairSigner.

```typescript
import { Account } from "@near-js/accounts"
import { JsonRpcProvider } from "@near-js/providers"
import { KeyPairSigner } from "@near-js/signers"

const provider = new JsonRpcProvider({
  url: "https://test.rpc.fastnear.com",
})
const signer = KeyPairSigner.fromSecretKey("ed25519:...")
const account = new Account("alice.testnet", provider, signer)
```

**near-kit:**
Configuration is flattened. The KeyStore is optional for simple cases.

```typescript
const near = new Near({
  network: "testnet",
  privateKey: "ed25519:...", // Automatically sets up an InMemoryKeyStore
  defaultSignerId: "alice.testnet",
})
```

### 2. Handling Units

**near-api-js:**
Requires manual conversion, often leading to `BN` (BigNumber) headaches.

```typescript
import { parseNearAmount } from "@near-js/utils"

const amount = parseNearAmount("10.5") // "1050000..."
const gas = "30000000000000" // Hope you counted the zeros right!
```

**near-kit:**
Parses human-readable strings automatically.

```typescript
const amount = "10.5 NEAR"
const gas = "30 Tgas"
```

### 3. Calling Contracts

**near-api-js:**
Arguments are passed inside a configuration object.

```typescript
const account = new Account("alice.testnet", provider, signer)

await account.callFunction({
  contractId: "market.near",
  methodName: "buy",
  args: { id: "1" },
  gas: "50000000000000",
  deposit: parseNearAmount("1")!,
})
```

**near-kit:**
Uses a fluent chain.

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

### 4. Error Handling

**near-api-js:**
Often throws raw RPC errors or generic "TypedErrors" that are hard to parse.

```typescript
try {
  // ...
} catch (e) {
  // You have to inspect e.type or e.message string matching
  if (e.type === 'FunctionCallError') { ... }
}
```

**near-kit:**
Throws distinct, standard JavaScript Error subclasses.

```typescript
import { FunctionCallError } from "near-kit"

try {
  // ...
} catch (e) {
  if (e instanceof FunctionCallError) {
    console.log(e.panic) // Access the panic message directly
  }
}
```

### 5. Access Keys

**near-api-js:**

```typescript
await account.addFunctionCallAccessKey({
  publicKey,
  contractId: "market.near",
  methodNames: ["buy"],
  allowance: parseNearAmount("0.25")!,
})
```

**near-kit:**
Explicitly typed permissions object.

```typescript
await near
  .transaction("alice.testnet")
  .addKey(publicKey, {
    type: "functionCall",
    receiverId: "market.near",
    methodNames: ["buy"],
    allowance: "0.25 NEAR",
  })
  .send()
```
