# AI Agent Cookbook: Introduction to near-kit

This document provides essential introductory information for an AI agent using `near-kit`.

## `near-kit` is NOT `near-api-js`

A critical point of understanding is that `near-kit` is a new, modern library for interacting with the NEAR blockchain. It is **not** `near-api-js` or a part of the `@near-js/*` family of libraries.

Many existing codebases and training datasets are based on `near-api-js`. While the core concepts of NEAR are the same, the APIs are fundamentally different. An agent trained on `near-api-js` must adapt to the new patterns in `near-kit`.

### Key Differences from `near-api-js`

- **Simplicity & Fluency**: `near-kit` uses a fluent, chainable API for building transactions, which is more readable and less verbose.
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
    { gas: "50 Tgas", attachedDeposit: "10 NEAR" }
  )
  .send();
```

An agent should **never** assume `near-api-js` patterns, such as `account.viewFunction` or `utils.format.parseNearAmount`, will work. It should instead rely on the patterns documented in this cookbook, such as `near.view()` and the `near.transaction()` builder.

### Human-Readable Units

`near-kit` accepts human-readable strings directly for amounts and gas:

```typescript
// Use string literals with units
await near.transaction("alice.testnet").transfer("bob.testnet", "10.5 NEAR").send();

// Or use the Amount helper for dynamic values
import { Amount } from "near-kit";
const deposit = Amount.NEAR(10.5); // "10.5 NEAR"
await near.transaction("alice.testnet").transfer("bob.testnet", deposit).send();
```

For displaying balances, use `formatAmount()`:

```typescript
import { formatAmount } from "near-kit";

const accountDetails = await near.getAccountDetails("alice.testnet");
const formatted = formatAmount(accountDetails.amount); // "100.50 NEAR"

// Customize formatting
const custom = formatAmount(accountDetails.amount, {
  precision: 4,        // Decimal places (default: 2)
  trimZeros: true,     // Remove trailing zeros (default: false)
  includeSuffix: false // Exclude " NEAR" suffix (default: true)
});
```

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
