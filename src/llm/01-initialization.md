# AI Agent Cookbook: Initialization

This document provides patterns for initializing the `near-kit` `Near` class for an AI agent.

## Core Concepts

- The `Near` class is the main entry point.
- Configuration is passed as an object to the constructor.
- The most important options are `network` and a credential source (`privateKey`, `keyStore`, or `wallet`).

## Type Definition: `NearConfig`

```typescript
import type { KeyStore, Signer, WalletConnection } from "near-kit";

export type NearConfig = {
  // Network selection
  network?: "mainnet" | "testnet" | "localnet" | {
    networkId: string;
    rpcUrl: string;
    walletUrl?: string;
  };
  
  // Optional RPC override and custom headers
  rpcUrl?: string;
  headers?: Record<string, string>;

  // Credential source (choose one)
  keyStore?: KeyStore | Record<string, string>;
  signer?: Signer;
  privateKey?: `ed25519:${string}` | `secp256k1:${string}`;
  wallet?: WalletConnection;

  // Default account for signing
  defaultSignerId?: string;

  // Default finality for transactions
  defaultWaitUntil?: "NONE" | "INCLUDED" | "EXECUTED_OPTIMISTIC" | "FINAL";
  
  // Retry behavior for network errors
  retryConfig?: {
    maxRetries?: number;
    initialDelayMs?: number;
  };
};
```

---

## Patterns

### Pattern 1: Connect to Testnet with a Private Key

This is the most common pattern for server-side scripts and testing.

```typescript
import { Near } from "near-kit";

// Initialize for testnet with a specific private key and default signer account
const near = new Near({
  network: "testnet",
  privateKey: "ed25519:...", // Replace with a valid ed25519 private key
  defaultSignerId: "my-account.testnet",
});
```

### Pattern 2: Connect to Mainnet (Read-Only)

To perform read-only operations (like `view` calls), no credentials are needed.

```typescript
import { Near } from "near-kit";

// Initialize for mainnet for read-only operations
const near = new Near({
  network: "mainnet",
});
```

### Pattern 3: Using an In-Memory Key Store

Use `InMemoryKeyStore` to manage multiple keys for the lifetime of the process.

```typescript
import { Near, InMemoryKeyStore } from "near-kit";

// Pre-populate a key store with multiple accounts
const keyStore = new InMemoryKeyStore({
  "alice.testnet": "ed25519:...",
  "bob.testnet": "ed25519:...",
});

const near = new Near({
  network: "testnet",
  keyStore,
});

// Transactions can now be signed by 'alice.testnet' or 'bob.testnet'
// e.g., near.transaction("alice.testnet")...
```

### Pattern 4: Using a File-Based Key Store (Node.js only)

Use `FileKeyStore` to access keys stored in the `near-cli` compatible format (`~/.near-credentials`).

```typescript
import { Near } from "near-kit";
import { FileKeyStore } from "near-kit/keys";

// This will read keys from the ~/.near-credentials/testnet directory
const keyStore = new FileKeyStore("~/.near-credentials", "testnet");

const near = new Near({
  network: "testnet",
  keyStore,
});
```

### Pattern 5: Using a Custom RPC with an API Key

If you are using a third-party RPC provider that requires an API key, use the `rpcUrl` and `headers` options.

```typescript
import { Near } from "near-kit";

const near = new Near({
  network: "mainnet",
  rpcUrl: "https://my.rpc.provider.com/mainnet",
  headers: {
    "x-api-key": "MY_API_KEY",
  },
});
```
