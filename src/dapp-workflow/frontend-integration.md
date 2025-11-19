# Frontend Integration

`near-kit` works exactly the same in the browser as it does on the server. The only difference is how you handle credentials: instead of a private key, you use a **Wallet Adapter**.

## The Adapter Pattern

You initialize the `Near` instance with a `wallet` object. `near-kit` provides adapters for popular wallet libraries.

### Using NEAR Wallet Selector

[`@near-wallet-selector`](https://github.com/near/wallet-selector) is a popular library for connecting wallets.

1.  **Setup Wallet Selector** (standard setup).
2.  **Get the wallet object**.
3.  **Wrap it with `fromWalletSelector`**.

```typescript
import { setupWalletSelector } from "@near-wallet-selector/core";
import { Near, fromWalletSelector } from "near-kit";

// 1. Setup selector
const selector = await setupWalletSelector({ ... });

// 2. User logs in... get the wallet
const wallet = await selector.wallet();

// 3. Initialize Near with the adapter
const near = new Near({
  network: "testnet",
  wallet: fromWalletSelector(wallet)
});

// Now use 'near' just like on the server!
// The wallet popup will appear for the user to approve.
await near.send("bob.near", "1 NEAR");
```

### Using HOT Connector

[HOT Connector](https://github.com/azbang/hot-connector) is the new standard for connecting wallets.

```typescript
import { NearConnector } from "@hot-labs/near-connect";
import { Near, fromHotConnect } from "near-kit";

const connector = new NearConnector({ ... });

connector.on("wallet:signIn", async () => {
  const near = new Near({
    network: "testnet",
    wallet: fromHotConnect(connector)
  });

  // Ready to transact
});
```
