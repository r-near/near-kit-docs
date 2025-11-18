# Frontend Integration

While server-side applications use [private keys](../core-concepts/key-management.md), [decentralized applications (dApps)](../dapp-development/message-signing.md) in the browser need to interact with a user's wallet. `near-kit` provides simple adapters to integrate with the most popular wallet connection libraries.

```admonish info title="Looking for a complete example?"
A fully tested frontend application using `near-kit` with both [Wallet Selector](./wallet-selector.md) and [HOT Protocol](./hot-connector.md) is [available here](https://github.com/r-near/near-kit-guestbook-demo). It serves as a practical, real-world reference for the patterns described in this guide.
```

## The Universal Code Pattern

A key design principle of `near-kit` is that your **business logic should not change** whether you are running on a server or in a browser. The way you [build a transaction](../transactions/builder.md) is always the same.

```typescript
// This function works anywhere!
async function addGuestbookMessage(
  near: Near,
  signerId: string,
  message: string
) {
  return await near
    .transaction(signerId)
    .functionCall(
      "guestbook.near-examples.testnet",
      "add_message",
      { text: message },
      { gas: "30 Tgas" }
    )
    .send()
}
```

The only difference is how you initialize the [`Near` class](../core-concepts/near-instance.md).

- **Server:** `new Near({ privateKey: '...' })`
- **Browser:** `new Near({ wallet: fromWalletAdapter(...) })`
