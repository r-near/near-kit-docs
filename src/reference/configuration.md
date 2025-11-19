# Configuration Options

These are the properties accepted by the `Near` class constructor.

```typescript
const near = new Near({
  network: "testnet",
  privateKey: "...",
})
```

## Network

Controls which NEAR network to connect to.

- **`network`**: `string | NetworkConfig`

  - **Presets**: `"mainnet"`, `"testnet"`, `"localnet"`.
  - **Custom**:
    ```typescript
    {
      networkId: "my-private-net",
      rpcUrl: "http://127.0.0.1:3030"
    }
    ```
  - **Sandbox**: Passing a `Sandbox` instance automatically configures RPC and the root account key.

- **`rpcUrl`**: `string` - (Optional) Override the RPC URL for a preset network (e.g. to use a specific provider like Infura or Lava).
- **`headers`**: `Record<string, string>` - (Optional) Custom HTTP headers for RPC requests (e.g. `{ "Authorization": "Bearer ..." }`).

## Credentials & Signing

`near-kit` checks these in order: `wallet` > `signer` > `privateKey` > `keyStore`.

- **`privateKey`**: `string` - A single private key (`ed25519:...`). Useful for scripts.
- **`keyStore`**: `KeyStore` - A storage backend for multiple keys.
  - `InMemoryKeyStore`: Default.
  - `FileKeyStore`: Node.js only. Stores in `~/.near-credentials`.
  - `NativeKeyStore`: Node.js only. Stores in macOS Keychain / Windows Credential Manager.
  - `RotatingKeyStore`: For high-concurrency apps.
- **`wallet`**: `WalletAdapter` - A wrapper around a browser wallet (e.g. `fromWalletSelector(wallet)`).
- **`signer`**: `(hash: Uint8Array) => Promise<Signature>` - A custom signing function. Use this for hardware wallets, KMS, or multi-sig logic.
- **`defaultSignerId`**: `string` - The account ID to use when calling `near.transaction()` without arguments.

## Execution Behavior

- **`defaultWaitUntil`**: `"INCLUDED" | "EXECUTED_OPTIMISTIC" | "FINAL"`

  - Controls how long `.send()` waits before returning.
  - Default: `"EXECUTED_OPTIMISTIC"` (Transaction is executed and result is available).
  - `"FINAL"`: Wait for BFT finality (safest, slowest).
  - `"INCLUDED"`: Return as soon as the network accepts it (fastest, no result value).

- **`retryConfig`**: `object`
  - **`maxRetries`**: `number` - Max retries for retryable network errors (429, 503, etc). Default: `4`.
  - **`initialDelayMs`**: `number` - Backoff delay. Default: `1000`ms.
