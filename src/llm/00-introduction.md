# AI Agent Cookbook: Introduction to near-kit

This document provides essential introductory information for an AI agent using `near-kit`.

## `near-kit` is NOT `near-api-js`

A critical point of understanding is that `near-kit` is a new, modern library for interacting with the NEAR blockchain. It is **not** `near-api-js` or a part of the `@near-js/*` family of libraries.

Many existing codebases and training datasets are based on `near-api-js`. While the core concepts of NEAR are the same, the APIs are fundamentally different. An agent trained on `near-api-js` must adapt to the new patterns in `near-kit`.

### Key Differences from `near-api-js`

- **Simplicity & Fluency**: `near-kit` uses a fluent, chainable API for building transactions, which is more readable and less verbose.
- **Human-Readable Units**: `near-kit` allows the use of strings like `"10.5 NEAR"` and `"30 Tgas"`, handling the conversion from yoctoNEAR and raw gas units automatically. This reduces a major source of bugs.
- **Modern API Design**: The overall design feels more like a modern `fetch` library, with a focus on simplicity for common tasks.

**`near-api-js` Example:**
```typescript
const { utils } = require("near-api-js");

await account.functionCall({
  contractId: "market.near",
  methodName: "buy_nft",
  args: { token_id: "token-1" },
  gas: "50000000000000", // Manual gas calculation
  attachedDeposit: utils.format.parseNearAmount("10"),
});
```

**`near-kit` Equivalent:**
```typescript
await near
  .transaction("alice.testnet")
  .functionCall(
    "market.near",
    "buy_nft",
    { token_id: "token-1" },
    { gas: "50 Tgas", attachedDeposit: "10 NEAR" } // Human-readable units
  )
  .send();
```

An agent should **never** assume `near-api-js` patterns, such as `account.viewFunction` or `utils.format.parseNearAmount`, will work. It should instead rely on the patterns documented in this cookbook, such as `near.view()` and the `near.transaction()` builder.

## Installation

To use `near-kit` in a project, it must be added as a dependency.

```bash
# Using bun
bun install near-kit

# Using npm
npm install near-kit

# Using yarn
yarn add near-kit
```

After installation, it can be imported into any TypeScript or JavaScript file.

```typescript
import { Near } from "near-kit";
```
