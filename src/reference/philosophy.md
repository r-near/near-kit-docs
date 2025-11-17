# Design Philosophy

`near-kit` is designed to handle many of the sharp edges of blockchain development for you automatically. This lets you focus on your application's logic instead of on boilerplate and error handling. Two key areas where this philosophy is applied are nonce management and network retries.

## Automatic Nonce Management

On NEAR, every transaction must have a unique nonce (a sequentially increasing number) for a given access key. If you try to send two transactions from the same account at the same time, they might end up with the same nonce, causing one of them to fail with an `InvalidNonce` error.

Manually tracking nonces, especially in an asynchronous application, is complex and error-prone.

`near-kit` solves this completely. It maintains an internal nonce manager that:
1.  Fetches the current nonce for a key from the blockchain.
2.  Caches it in memory.
3.  Atomically increments the nonce for every new transaction built.
4.  Automatically detects `InvalidNonce` errors from the network.
5.  If a nonce error occurs, it automatically invalidates its cache, fetches the correct nonce from the chain, and retries the transaction for you.

This means you can safely do things that would normally be very difficult:

```typescript
// Send three transactions concurrently without worrying about nonces
await Promise.all([
  near.send("bob.near", "1"),
  near.send("charlie.near", "1"),
  near.send("dave.near", "1"),
]);

// All three transactions will get unique, sequential nonces and succeed.
```

You don't need to do anything to enable this feature—it's built into the `TransactionBuilder` and works out of the box.

## Smart Retry Logic

Interacting with a decentralized network of RPC nodes means that sometimes, requests will fail for transient reasons: a node might be temporarily overloaded, a network connection might drop, or a block might not have propagated yet.

`near-kit` includes a smart retry mechanism with exponential backoff for all RPC calls. If a request fails with a retryable error (like a `503 Service Unavailable` or a `429 Too Many Requests`), the library will automatically wait and try again several times before failing.

This makes your application more resilient to temporary network issues without requiring you to write complex retry loops.

```typescript
try {
  // This call might fail temporarily due to a network hiccup
  await near.call("contract.near", "method", {});
} catch (error) {
  // If the error is a TimeoutError and it's marked as retryable,
  // it means near-kit already tried several times for you.
  if (error instanceof TimeoutError && error.retryable) {
    console.log("Network is unstable, but near-kit already retried automatically.");
  }
}
```

You can customize the retry behavior in the `Near` constructor if needed, but the default settings are suitable for most applications.