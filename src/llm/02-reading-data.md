# AI Agent Cookbook: Reading Data

This document provides patterns for reading data from the NEAR blockchain using `near-kit`. These operations are read-only, do not cost gas, and do not require credentials.

## Core Concepts

- All read operations are `async` and return a `Promise`.
- Most read operations can take an optional `BlockReference` object to query historical data.

## Type Definition: `BlockReference`

```typescript
export type BlockReference = {
  /**
   * Finality level for the query.
   * 'optimistic': Latest block, possibly not final.
   * 'final': Fully finalized block.
   * Default is 'final' for view calls.
   */
  finality?: "optimistic" | "final";

  /**
   * A specific block ID (height or hash) to query against.
   * Overrides finality if provided.
   */
  blockId?: number | string;
};
```

---

## Patterns

### Pattern 1: Call a Contract View Method

Use `near.view()` to call a read-only method on a smart contract.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet" });

// Call 'get_messages' on a guestbook contract
const messages = await near.view(
  "guestbook.near-examples.testnet",
  "get_messages",
  {} // Arguments for the view call
);
// `messages` will contain the parsed JSON result from the contract.
```

### Pattern 2: Get an Account's NEAR Balance

Use `near.getBalance()` to retrieve the balance of an account.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet" });

// Get the balance for 'alice.testnet'
const balance = await near.getBalance("alice.testnet");
// `balance` will be a formatted string, e.g., "100.00 NEAR".
```

### Pattern 3: Check if an Account Exists

Use `near.accountExists()` for a boolean check.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet" });

// Check if 'alice.testnet' exists
const exists = await near.accountExists("alice.testnet");
// `exists` will be true or false.
```

### Pattern 4: Get Network Status

Use `near.getStatus()` to get high-level information about the network.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet" });

// Get the current network status
const status = await near.getStatus();
// `status` will be an object like:
// {
//   chainId: 'testnet',
//   latestBlockHeight: 123456789,
//   syncing: false
// }
```

### Pattern 5: Get Detailed Transaction Status

Use `near.getTransactionStatus()` to get the detailed outcome of a past transaction.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet" });

const txHash = "8ZQ7...your_tx_hash"; // A valid transaction hash
const senderId = "alice.testnet";     // The account that sent the transaction

// Get the final outcome of the transaction
const result = await near.getTransactionStatus(txHash, senderId, "FINAL");
// `result` will be a `FinalExecutionOutcomeWithReceipts` object.
```

### Pattern 6: Querying at a Specific Block Height

Most read methods accept a `BlockReference`. Use this to query historical state.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet" });

// Get the balance of 'alice.testnet' at a specific block height
const historicalBalance = await near.getBalance("alice.testnet", {
  blockId: 123450000,
});
```
