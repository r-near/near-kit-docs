# View vs. Call

On any blockchain, there are two fundamental types of operations: reading data and writing data. `near-kit` makes this distinction clear with `view()` and `call()`/`transaction()`.

## `near.view()` - Reading Data

- **Read-only:** It cannot change anything on the blockchain.
- **Free:** It costs no gas.
- **No Signature:** It doesn't require a private key.
- **Fast:** It returns the data you asked for directly.

Use `near.view()` whenever you want to query a contract's state.

```typescript
const messages = await near.view(
  "guestbook.near-examples.testnet",
  "get_messages",
  {}
)
```

## `near.transaction()` - Writing Data

- **State-Changing:** This is for any action that modifies the blockchain (sending tokens, calling a change method, etc.).
- **Costs Gas:** It requires the signer to pay a transaction fee.
- **Requires Signature:** It must be signed by a private key or a wallet.
- **Asynchronous:** It returns a transaction receipt, and the final result happens on-chain.

Use `near.transaction()` for any operation that writes data.

```typescript
const result = await near
  .transaction("alice.testnet")
  .functionCall("guestbook.near-examples.testnet", "add_message", {
    text: "Hello!",
  })
  .send()
```