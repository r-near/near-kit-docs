# Configuration

The `Near` class constructor accepts a configuration object that allows you to customize its behavior for different networks, credentials, and environments.

```typescript
import { Near } from "near-kit"

const near = new Near({
  // ... your configuration options
})
```

## Network Configuration

You can connect to a standard NEAR network or a custom one.

### Network Presets

Use a simple string for standard networks. The `NEAR_NETWORK` environment variable will be used if no network is specified. Defaults to `"mainnet"`.

- `"mainnet"`
- `"testnet"`
- `"localnet"`

```typescript
// Connect to the testnet
const near = new Near({ network: "testnet" })
```

### Custom Network

For custom networks (like a private shard or a unique RPC endpoint), provide a network object.

```typescript
const near = new Near({
  network: {
    networkId: "my-custom-net",
    rpcUrl: "https://my.rpc.node/proxy",
    walletUrl: "https://my.wallet.com",
  },
})
```

You can also override just the RPC URL while using a preset for the other defaults.

```typescript
// Use testnet defaults, but with a specific RPC provider
const near = new Near({
  network: "testnet",
  rpcUrl: "https://rpc.testnet.pagoda.co/rpc/v1/",
})
```

## Credential & Signer Configuration

This determines how transactions will be signed. The options are evaluated in the following order of priority:

1.  **`wallet`**: A wallet adapter for browser-based dApps. All signing is delegated to the user's wallet.
2.  **`signer`**: A custom asynchronous function for signing transaction hashes. Use this for integrating hardware wallets or external KMS.
3.  **`privateKey`**: A single private key string (e.g., `"ed25519:..."`) for signing all transactions.
4.  **`keyStore`**: An instance of a key store (`InMemoryKeyStore`, `FileKeyStore`, etc.) for managing multiple keys.

### `wallet`

For frontend applications, this property integrates `near-kit` with a browser wallet adapter. All signing operations are delegated to the user's connected wallet.

➡️ **See the [Frontend Integration](../frontend-integration/) guide for detailed instructions and examples.**

### `signer`

For advanced use cases like hardware wallets.

```typescript
const myHardwareSigner = async (message) => {
  // ... logic to sign with a hardware device
  return { keyType: KeyType.ED25519, data: signatureBytes }
}

const near = new Near({
  network: "testnet",
  signer: myHardwareSigner,
})
```

### `privateKey`

Simple and direct, ideal for scripts or server-side apps with a single identity.

```typescript
const near = new Near({
  network: "testnet",
  privateKey: process.env.MY_PRIVATE_KEY,
})
```

### `keyStore`

The most flexible option for managing multiple accounts. See the [Key Management](../core-concepts/key-management.md) guide for details on the different types of key stores.

```typescript
import { FileKeyStore } from "near-kit/keys"

const keyStore = new FileKeyStore("~/.near-credentials", "testnet")
const near = new Near({
  network: "testnet",
  keyStore,
})
```

## Other Options

### `defaultSignerId`

Specifies the account ID to use for signing when it's not otherwise specified in a method call.

```typescript
const near = new Near({
  network: "testnet",
  privateKey: "...",
  defaultSignerId: "my-app.testnet",
})

// This will be signed by 'my-app.testnet'
await near.send("bob.testnet", "1 NEAR")
```

### `defaultWaitUntil`

Sets the default finality level to wait for when sending a transaction. Can be one of `"NONE"`, `"INCLUDED"`, `"EXECUTED_OPTIMISTIC"`, or `"FINAL"`.

Defaults to `"EXECUTED_OPTIMISTIC"`.

```typescript
const near = new Near({
  network: "testnet",
  privateKey: "...",
  defaultWaitUntil: "FINAL", // Wait for full finality on all transactions
})
```

### `headers`

Provide custom headers to be sent with every RPC request. This is useful for services that require an API key.

```typescript
const near = new Near({
  network: "mainnet",
  headers: {
    "x-api-key": "MY_RPC_PROVIDER_API_KEY",
  },
})
```

### `retryConfig`

Customize the automatic retry behavior for network errors.

```typescript
const near = new Near({
  network: "mainnet",
  retryConfig: {
    maxRetries: 5, // Number of retries on failure
    initialDelayMs: 500, // Initial delay before first retry
  },
})
```
