# The `Near` Instance

The `Near` class is the main entry point for all interactions. You initialize it once with your configuration, and it can be reused throughout your application.

```typescript
import { Near } from "near-kit"

const near = new Near({
  network: "testnet",
  privateKey: "ed25519:...",
})
```

The constructor is flexible and can be configured for different networks, key management strategies, and environments.

## Common Configuration Options

Here are some of the most common ways to configure the `Near` instance.

### Network

You can connect to a standard NEAR network or a custom RPC endpoint.

```typescript
// Connect to a standard network (mainnet, testnet, localnet)
const near = new Near({ network: "testnet" })

// Connect to a custom RPC endpoint
const nearWithCustomRpc = new Near({
  network: "testnet", // Still useful for defaults
  rpcUrl: "https://rpc.testnet.pagoda.co/rpc/v1/",
})
```

### Credentials

To [sign transactions](../transactions/builder.md), you must provide credentials. `near-kit` supports several methods, which are prioritized in the following order:

1.  **[`wallet`](../frontend-integration/)**: For browser apps, this delegates signing to a user's wallet.
2.  **`signer`**: A custom async function to handle signing, for hardware wallets or external KMS.
3.  **`privateKey`**: A single private key string, ideal for scripts.
4.  **`keyStore`**: A flexible key store for managing one or more keys.

```typescript
// Using a single private key (common for servers/scripts)
const near = new Near({
  network: "testnet",
  privateKey: process.env.MY_PRIVATE_KEY,
  defaultSignerId: "my-app.testnet",
})

// Using a Key Store (manages multiple keys)
import { InMemoryKeyStore } from "near-kit"
const keyStore = new InMemoryKeyStore({
  "alice.testnet": "ed25519:...",
  "bob.testnet": "ed25519:...",
})
const nearWithKeyStore = new Near({
  network: "testnet",
  keyStore,
})
```

➡️ **See the [Key Management](./key-management.md) guide for a full overview of the different key store types.**

### Default Signer

If you provide a `privateKey` or a `keyStore` with multiple keys, you should specify which account to use for signing by default. This is used for simple write operations like [`near.send()`](./writing-data.md), which can take [human-readable units](./units.md) like `"1 NEAR"`.

```typescript
const near = new Near({
  network: "testnet",
  privateKey: "...",
  defaultSignerId: "my-app.testnet",
});

// This transaction will be signed by 'my-app.testnet'
await [near.send](./writing-data.md)("another-account.testnet", "[1 NEAR](./units.md)");
```

### Custom RPC Headers

If your RPC provider requires an API key, you can provide it via the `headers` option.

```typescript
const near = new Near({
  network: "mainnet",
  headers: {
    "x-api-key": "MY_RPC_PROVIDER_API_KEY",
  },
})
```

➡️ **See the [Configuration](../reference/configuration.md) reference for a complete list of all available options.**
