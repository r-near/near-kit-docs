# Delegate Actions (Meta-Transactions)

Delegate Actions, officially specified in [NEP-366](https://github.com/near/NEPs/pull/366), are a powerful feature that allows one account (a "user") to sign a set of [actions](../reference/actions.md) that another account (a "relayer") can execute and pay for on their behalf. This enables "meta-transactions," creating gasless experiences for end-users where a backend service or another user sponsors the transaction fees.

`near-kit` provides a simple and secure way to create and submit these delegate actions.

## The Core Idea

1.  **User Creates Actions**: The user decides what they want to do (e.g., call a contract method, transfer tokens).
2.  **User Signs Off-Chain**: The user signs these actions with their key, but **does not** send them to the blockchain. This signature is free and happens off-chain. The result is an object containing the structured `SignedDelegateAction` plus an encoded `payload` that is safe to transport.
3.  **User Sends to Relayer**: The user sends the `payload` (a base64 string by default) to a relayer service (e.g., via a standard HTTPS POST request). The structured data can still be used locally for wallet UI/validation.
4.  **Relayer Submits On-Chain**: The relayer decodes the payload back into a `SignedDelegateAction`, wraps it in its own [transaction](../transactions/builder.md), signs the transaction with its own key, and submits it to the blockchain. The relayer pays the [gas fee](../core-concepts/units.md).
5.  **Blockchain Verification**: The NEAR protocol verifies both the relayer's signature and the original user's signature before executing the actions as the user.

## Creating a Delegate Action

The user-facing part of the flow is creating and signing the delegate action. This is done using the [`transaction()` builder](../transactions/builder.md) followed by the `.delegate()` method instead of `.send()`.

```typescript
import { Near } from "near-kit"

// User's client, configured with their private key or wallet
const userNear = new Near({
  network: "testnet",
  privateKey: "ed25519:...", // User's private key
  defaultSignerId: "user.testnet",
})

// 1. User builds a transaction with the actions they want to perform
const transaction = userNear
  .transaction("user.testnet")
  .functionCall("guestbook.testnet", "add_message", {
    text: "Hello from a meta-tx!",
  })

// 2. Instead of .send(), the user calls .delegate() to sign it off-chain
const { signedDelegateAction, payload, format } = await transaction.delegate()

// 3. Send the encoded payload (base64 by default) to a relayer's backend API
await fetch("https://my-relayer.com/submit", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ payload }),
})

console.log("Structured action for UI/logging:", signedDelegateAction)
console.log("Encoded payload format:", format)

// Need raw bytes instead? call .delegate({ payloadFormat: "bytes" })
```

The `signedDelegateAction` object contains all the information a wallet or UI needs, while the encoded `payload` is safe to ship over HTTP. Pass `{ payloadFormat: "bytes" }` when calling `.delegate()` if you prefer a raw `Uint8Array` for binary transports.

## Submitting a Delegate Action (Relayer)

The relayer's job is to take the `SignedDelegateAction` from the user, wrap it in a new transaction, and send it.

```typescript
import { Near, decodeSignedDelegateAction } from "near-kit"

// Relayer's client, configured with its own key for paying gas
const relayerNear = new Near({
  network: "testnet",
  privateKey: "ed25519:...", // Relayer's private key
  defaultSignerId: "relayer.testnet",
})

// Assume this is an HTTP endpoint that receives the payload from the user
async function handleSubmit(payload: string) {
  console.log(
    "Received delegate action from user, submitting to the network..."
  )

  const signedDelegate = decodeSignedDelegateAction(payload)

  // The relayer creates a transaction where its own account is the signer
  const result = await relayerNear
    .transaction("relayer.testnet")
    // The .signedDelegateAction() method is used to wrap the user's action
    .signedDelegateAction(signedDelegate)
    .send()

  console.log("✅ Relayer successfully submitted transaction!")
  console.log("Transaction Hash:", result.transaction.hash)

  return result
}
```

When this transaction is executed, the `add_message` function on the `guestbook.testnet` contract will see `user.testnet` as the `predecessor_account_id`, even though `relayer.testnet` paid for the gas.

```admonish info title="Transport formats"
`.delegate()` returns `{ payload, format, signedDelegateAction }`. The default format is `"base64"`, which is safe for JSON/HTTP. Pass `{ payloadFormat: "bytes" }` when calling `.delegate()` to receive a `Uint8Array` payload instead.
```

## Security Considerations

- **Expiration**: Delegate actions expire after a certain block height to prevent them from being replayed indefinitely. By default, `near-kit` sets this to 200 blocks from the current height. You can customize this with the `maxBlockHeight` or `blockHeightOffset` options in the `.delegate()` method.
- **Nonce**: Each delegate action has a nonce, just like a regular transaction. `near-kit` automatically fetches and increments the correct nonce for the user's [access key](../core-concepts/key-management.md), preventing replay attacks.
- **Receiver ID**: The actions within the delegate action are executed with a specific `receiverId`. The relayer cannot change what contract the user is interacting with.

```admonish tip title="Key Security Features"
- **Expiration**: Delegate actions are only valid for a limited time.
- **Nonce Management**: `near-kit` handles nonces automatically to prevent replay attacks.
- **Receiver Integrity**: The contract being called cannot be changed by the relayer.
```
