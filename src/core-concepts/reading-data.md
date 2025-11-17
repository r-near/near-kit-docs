# Reading Data from the Blockchain

`near-kit` provides several convenient, one-line methods for reading data from the NEAR blockchain. All of these are read-only operations, which means they are fast, free, and do not require a signature.

## Calling a View Method

The most common way to read data is by calling a `view` method on a smart contract. This is done using the `near.view()` method.

```typescript
// Get the latest messages from a guestbook contract
const messages = await near.view(
  "guestbook.near-examples.testnet",
  "get_messages",
  {} // Arguments, if any
)
```

For more details on view calls, see the [View vs. Call](./view-vs-call.md) guide.

## Checking an Account's Balance

You can get an account's NEAR balance using `near.getBalance()`.

```typescript
const balance = await near.getBalance("alice.testnet")

console.log(balance) // e.g., "100.00 NEAR"
```

The balance is returned as a formatted string with two decimal places and the " NEAR" suffix.

````admonish note title="Getting the Raw Balance"
The `near.getBalance()` method is convenient for display purposes. If you need the raw balance in [yoctoNEAR](./units.md) for calculations, use `near.getAccountDetails()` instead.

```typescript
const details = await near.getAccountDetails("alice.testnet");
console.log(details.amount); // e.g., "100000000000000000000000000" (a string in yoctoNEAR)
```
````

## Checking if an Account Exists

To see if an account has been created on the network, use `near.accountExists()`.

```typescript
const exists = await near.accountExists("nonexistent.testnet")

console.log(exists) // false
```

This is a simple boolean check that is useful for validation before sending funds or creating sub-accounts.

## Getting Network Status

You can query the basic status of the network with `near.getStatus()`.

```typescript
const status = await near.getStatus()

console.log(status)
// {
//   chainId: 'testnet',
//   latestBlockHeight: 123456789,
//   syncing: false
// }
```

This is useful for checking network health or getting the latest block height.

## Querying Transaction Status

When you [send a transaction](../transactions/builder.md), you get a hash back. To check the final result of that transaction, especially in asynchronous workflows, you can use `near.getTransactionStatus()`. This method provides a detailed breakdown of the transaction's execution, including receipts, logs, and potential failures.

```typescript
const txHash = "8ZQ7...your_tx_hash"
const senderId = "alice.testnet"

const result = await near.getTransactionStatus(txHash, senderId)
```

This method is essential for tracking the result of a transaction. Here’s how to inspect the result:

### Checking for Success or Failure

The `status` field tells you if the transaction succeeded. If it failed, the `Failure` property will contain details.

```typescript
if ("Failure" in result.status) {
  console.error("Transaction failed:", result.status.Failure)
} else {
  console.log("Transaction succeeded!")
}
```

### Checking for Contract Panics

A common failure case is a smart contract panic. You can find the panic message by inspecting the `receipts_outcome`.

```typescript
for (const receipt of result.receipts_outcome) {
  if (
    typeof receipt.outcome.status === "object" &&
    "Failure" in receipt.outcome.status
  ) {
    // This is a [FunctionCallError](./error-handling.md)
    const failure = receipt.outcome.status.Failure
    console.error(`Contract panicked in receipt ${receipt.id}:`, failure)
    // You can now inspect the failure object for the specific error message.
  }
}
```

➡️ For more robust failure analysis, see the [Error Handling](./error-handling.md) guide.

### Getting the Return Value

If a contract method returns a value, it will be in `result.status.SuccessValue` as a base64-encoded string. You need to decode and parse it.

```typescript
if ("SuccessValue" in result.status && result.status.SuccessValue) {
  const returnValueBase64 = result.status.SuccessValue
  const decodedValue = Buffer.from(returnValueBase64, "base64").toString()

  try {
    const parsedValue = JSON.parse(decodedValue)
    console.log("Contract returned:", parsedValue)
  } catch (e) {
    console.log("Contract returned a raw string:", decodedValue)
  }
}
```

## Batching Read Operations

If you need to make multiple read calls at once, you can use `near.batch()` to run them in parallel. This is a simple convenience wrapper around `Promise.all` that preserves the tuple typing of the results.

```typescript
const [balance, status, exists] = await near.batch(
  near.getBalance("alice.testnet"),
  near.getStatus(),
  near.accountExists("bob.testnet")
)

console.log(
  `Balance: ${balance}, Synced: ${!status.syncing}, Bob exists: ${exists}`
)
```
