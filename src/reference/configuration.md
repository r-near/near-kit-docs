# Configuration Options

Properties accepted by the `Near` constructor.

```typescript
type NearConfig = {
  // Network (Default: 'mainnet')
  network?: "mainnet" | "testnet" | "localnet" | Sandbox

  // Credentials (Choose one)
  privateKey?: string // 'ed25519:...'
  keyStore?: KeyStore // InMemory, File, Rotating, etc.
  wallet?: WalletAdapter // fromWalletSelector()
  signer?: SignerFunction // Custom signer

  // Defaults
  defaultSignerId?: string // Used if .transaction() signer is omitted
  defaultWaitUntil?: "INCLUDED" | "EXECUTED_OPTIMISTIC" | "FINAL"

  // Advanced
  rpcUrl?: string // Override default RPC
  headers?: Record<string, string> // Custom headers (e.g. API keys)
  retryConfig?: {
    // Customize retry logic
    maxRetries?: number
    initialDelayMs?: number
  }
}
```
