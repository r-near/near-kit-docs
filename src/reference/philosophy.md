# Design Philosophy

Blockchain libraries often force you to think about things that shouldn't matter: which JavaScript runtime you're using, how to construct action arrays, or whether an error has the property you need. `near-kit` takes a different approach. It's designed around a few core principles that make working with NEAR feel closer to regular JavaScript development.

## Write Once, Run Anywhere

If you've used `near-api-js` before, you've probably written different code for different environments. Browser apps needed wallet adapters and special setup. Server code used a different pattern. Edge workers and React Native required their own workarounds.

`near-kit` works the same everywhere. Here's a simple example:

```typescript
import { Near } from "near-kit"

const near = new Near({
  network: "testnet",
  privateKey: "ed25519:...",
})

await near.send("alice.testnet", "1")
```

This code runs identically in Node.js, browsers, Cloudflare Workers, Vercel Edge, React Native, Deno, and Bun. There's no environment detection, no conditional imports, and no separate packages.

If you write a transaction builder in your backend and realize you need the same logic in your frontend, you can just import the same module. If you want to move a server function to an edge worker, you can deploy it without changes.

## The Transaction Builder Pattern

Transactions in `near-kit` use a chainable builder pattern. Instead of constructing arrays of actions or passing multiple parameters in a specific order, you build up the transaction step by step:

```typescript
// With near-api-js, you might write:
const actions = [
  transactions.functionCall(
    "increment",
    {},
    "30000000000000",
    "0"
  ),
]
await account.signAndSendTransaction({
  receiverId: "counter.testnet",
  actions,
})

// With near-kit:
await near
  .transaction("signer.testnet")
  .functionCall("counter.testnet", "increment", {})
  .send()
```

This becomes especially helpful when you're building complex transactions with multiple actions:

```typescript
await near
  .transaction("alice.testnet")
  .transfer("bob.testnet", "5")
  .functionCall("contract.testnet", "register", { name: "Alice" })
  .functionCall("contract.testnet", "notify", { event: "signup" })
  .send()
```

You can read this transaction like a sentence: Alice sends 5 NEAR to Bob, then calls `register` on the contract, then calls `notify`. Each action is clear, and the order is explicit. Because the API is chainable, your editor can autocomplete the available methods at each step.

## Type Safety as a Feature

`near-kit` is written in TypeScript, and the type system is designed to catch mistakes before you run your code. If you pass the wrong type of argument or forget a required parameter, your editor will show an error immediately:

```typescript
// TypeScript catches these at compile time:
await near.send("alice.testnet", 123) // Error: expected string

await near.transaction() // Error: accountId required

const near = new Near({ network: "production" }) // Error: invalid network
```

Error handling is also type-safe. When you check an error with `instanceof`, TypeScript knows which properties are available:

```typescript
try {
  await near.call("contract.testnet", "method", {})
} catch (error) {
  if (error instanceof FunctionCallError) {
    // TypeScript knows these properties exist
    console.error(error.panic)        // string
    console.error(error.logs)         // string[]
    console.error(error.contractId)   // string
  } else if (error instanceof NetworkError) {
    // Different properties for different error types
    console.error(error.retryable)    // boolean
  }
}
```

This means you don't need to cast errors to `any` to access their properties, and you won't accidentally access a property that doesn't exist on that error type.

## Errors You Can Actually Handle

When something goes wrong on NEAR, the error you get should tell you what happened and give you enough context to handle it. `near-kit` provides typed error classes that include relevant details:

```typescript
try {
  await near
    .transaction("alice.testnet")
    .functionCall("game.testnet", "attack", { target: "bob" })
    .send()
} catch (error) {
  if (error instanceof FunctionCallError) {
    // The contract panicked. The error includes:
    console.error(`Contract: ${error.contractId}`)
    console.error(`Method: ${error.methodName}`)
    console.error(`Panic: ${error.panic}`)
    console.error(`Logs:`, error.logs)

    // If the panic message is something like "ERR_NOT_ENOUGH_MANA",
    // you can show a helpful message to your user
  }
}
```

Each error type includes the information you'd need to debug or handle that specific case:

- **`FunctionCallError`**: Includes the contract ID, method name, panic message, and any logs the contract emitted before failing.
- **`AccountDoesNotExistError`**: Includes the account ID that doesn't exist.
- **`NetworkError`**: Includes a `retryable` boolean so you know whether it makes sense to try again.
- **`InvalidTransactionError`**: Includes the specific reason the transaction was rejected.

## Under the Hood

While the API stays simple, `near-kit` handles some of the more complex blockchain details automatically.

### Automatic Nonce Management

On NEAR, every transaction must have a unique nonce (a sequentially increasing number) for a given access key. If you try to send two transactions from the same account at the same time, they might end up with the same nonce, causing one of them to fail with an `InvalidNonce` error.

`near-kit` maintains an internal nonce manager that:
1. Fetches the current nonce for a key from the blockchain.
2. Caches it in memory.
3. Atomically increments the nonce for every new transaction.
4. Detects `InvalidNonce` errors from the network.
5. If a nonce error occurs, invalidates its cache, fetches the correct nonce, and retries the transaction.

This means you can safely send multiple transactions concurrently:

```typescript
await Promise.all([
  near.send("bob.testnet", "1"),
  near.send("charlie.testnet", "1"),
  near.send("dave.testnet", "1"),
])
```

All three transactions will get unique, sequential nonces and succeed.

#### High-Throughput Scenarios

For applications that need to send dozens or hundreds of concurrent transactions from a single account, consider using [`RotatingKeyStore`](../core-concepts/key-management.md#rotatingkeystore). It eliminates nonce collisions entirely by rotating through multiple access keys, each with independent nonce tracking. This provides:

- **100% success rate** for concurrent transactions (no retries needed)
- **Better throughput** for high-volume operations
- **No race conditions** between concurrent transaction builds

Learn more in the [Key Management](../core-concepts/key-management.md#rotatingkeystore) guide.

### Smart Retry Logic

Interacting with a decentralized network means that sometimes, requests will fail for transient reasons: a node might be temporarily overloaded, a network connection might drop, or a block might not have propagated yet.

`near-kit` includes a retry mechanism with exponential backoff for all RPC calls. If a request fails with a retryable error (like `503 Service Unavailable` or `429 Too Many Requests`), the library will wait and try again several times before failing.

This makes your application more resilient to temporary network issues without requiring you to write retry loops yourself.
