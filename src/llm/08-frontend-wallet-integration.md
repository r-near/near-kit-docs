# AI Agent Cookbook: Frontend Wallet Integration

For decentralized applications (dApps) that run in a user's browser, transactions must be signed by a wallet. `near-kit` integrates with browser wallets by using a `wallet` adapter in the `Near` class constructor. This delegates all signing operations to the user's wallet.

## Core Concept: The `wallet` Adapter

Instead of providing a `privateKey` or `keyStore`, you initialize the `Near` instance with a `wallet` object. This object is created by an adapter function that bridges `near-kit` with a specific wallet connection library.

```typescript
// Server-side or script initialization
const nearOnServer = new Near({
  network: "testnet",
  privateKey: "ed25519:...",
});

// Browser-based dApp initialization
import { fromWalletSelector } from "near-kit";
import { setupWalletSelector } from "@near-wallet-selector/core";

// (Wallet Selector setup code...)
const wallet = await walletSelector.wallet();

const nearInBrowser = new Near({
  network: "testnet",
  wallet: fromWalletSelector(wallet), // The wallet adapter is passed here
});
```

## The Universal Code Pattern

A key design principle of `near-kit` is that your application's business logic should remain the same, regardless of the environment. The way you build a transaction is identical whether it's signed by a private key on a server or by a wallet in a browser.

This function for adding a message to a guestbook will work anywhere:
```typescript
// This function is universal and works in any environment
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
    .send();
}
```

The only difference is how the `near` instance passed to this function was initialized.

## Pattern 1: Using NEAR Wallet Selector

[`@near-wallet-selector`](https://github.com/near/wallet-selector) is the most common library for connecting to various NEAR wallets. `near-kit` integrates with it via the `fromWalletSelector` adapter.

### Flow:
1.  Set up `@near-wallet-selector/core` and desired wallet modules (e.g., `@near-wallet-selector/meteor-wallet`).
2.  When the user signs in, get the `wallet` object from the selector.
3.  Create a `near-kit` `Near` instance using `fromWalletSelector(wallet)`.
4.  Use this `near` instance to build and send transactions. The wallet selector will prompt the user for approval.

### Example:
```typescript
import { setupWalletSelector } from '@near-wallet-selector/core';
import { Near, fromWalletSelector } from 'near-kit';

// 1. Setup wallet selector
const walletSelector = await setupWalletSelector({
  network: 'testnet',
  modules: [/* ... wallet modules ... */],
});

// 2. Wait for user to sign in and get the wallet object
const wallet = await walletSelector.wallet();

if (wallet) {
  // 3. Create the Near instance with the adapter
  const near = new Near({
    network: 'testnet',
    wallet: fromWalletSelector(wallet),
  });

  const accountId = (await wallet.getAccounts())[0].accountId;

  // 4. Call the universal function
  await addGuestbookMessage(near, accountId, "Hello from Wallet Selector!");
}
```

## Pattern 2: Using HOT Connector

[`@hot-labs/near-connect`](https://github.com/azbang/hot-connector) is used for integrating with the HOT Wallet inside Telegram. `near-kit` supports this through the `fromHotConnect` adapter.

### Flow:
1.  Initialize the `NearConnector` from `@hot-labs/near-connect`.
2.  Listen for the `wallet:signIn` event.
3.  When the event fires, create a `near-kit` `Near` instance using `fromHotConnect(connector)`.
4.  Use this `near` instance to build and send transactions.

### Example:
```typescript
import { NearConnector } from "@hot-labs/near-connect";
import { Near, fromHotConnect } from "near-kit";

// 1. Initialize the HOT connector
const connector = new NearConnector({ network: "testnet" });

// 2. Listen for sign-in
connector.on("wallet:signIn", async (data) => {
  // 3. Create the Near instance with the adapter
  const near = new Near({
    network: "testnet",
    wallet: fromHotConnect(connector as any),
  });

  const accountId = data.accounts[0].accountId;

  // 4. Call the universal function
  await addGuestbookMessage(near, accountId, "Hello from HOT Wallet!");
});

// To trigger the flow, you would call:
// await connector.connect();
```
