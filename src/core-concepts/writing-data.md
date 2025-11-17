# Simple Write Operations

For the most common write operations—sending NEAR and calling a single contract method—`near-kit` provides simple, one-line methods that act as shortcuts to the more powerful [transaction builder](../transactions/builder.md).

These methods require a signer to be configured on the [`Near` instance](./near-instance.md), either via a [`privateKey`](./key-management.md), [`keyStore`](./key-management.md), or a [`wallet` adapter](../frontend-integration/).

## Sending NEAR

The `near.send()` method is the simplest way to transfer NEAR from the signer's account to a receiver.

- `receiverId: string`: The account that will receive the tokens.
- `amount: Amount`: The amount of NEAR to send (e.g., `"5 NEAR"`).

```typescript
// The default signer (from the Near constructor) sends 5 NEAR to bob.testnet
await near.send("bob.testnet", "[`5 NEAR`](./units.md)")
```

This is equivalent to using the transaction builder for a single [`transfer`](../reference/actions.md#transfer) action:

```typescript
await near
  .transaction(signerId)
  .transfer("bob.testnet", "[`5 NEAR`](./units.md)")
  .send()
```

## Calling a Change Method

The `near.call()` method is a direct way to execute a single change method on a smart contract.

- `contractId: string`: The contract to call.
- `methodName: string`: The name of the method to execute.
- `args?: object | Uint8Array`: The arguments for the method.
- `options?: CallOptions`: Options like [`gas`](./units.md), [`attachedDeposit`](./units.md), and `signerId`.

```typescript
// Call the 'increment' method on a counter contract
await near.call("counter.testnet", "increment", {})

// Call a method with arguments and an attached deposit
await near.call(
  "market.testnet",
  "buy",
  { nft_id: "token1" },
  { attachedDeposit: "[`10 NEAR`](./units.md)" }
)
```

This is a shortcut for a transaction with a single [`functionCall`](../reference/actions.md#functioncall) action.

### Specifying a Signer

If you have multiple keys in your key store, you can specify which account should sign the call using the `signerId` option.

```typescript
// Bob signs the call, even if Alice is the default signer
await near.call(
  "some-contract.testnet",
  "some_method",
  {},
  { signerId: "bob.testnet" }
)
```

## When to Use the Transaction Builder

While these shortcuts are convenient, you should use the full [Transaction Builder](../transactions/builder.md) when you need to:

- Perform multiple actions atomically (e.g., create an account AND transfer funds to it). See the `TransactionBuilder` docs for important details on the [boundaries of atomicity](../transactions/builder.md#atomic-by-nature-within-a-transaction).
- Use more advanced actions like [`addKey`](../reference/actions.md#addkey) or [`deployContract`](../reference/actions.md#deploycontract).
- Create meta-transactions using [`.delegate()`](../dapp-development/delegate-actions.md).
