# Human-Readable Units

One of the biggest sources of bugs in blockchain development is unit conversion. NEAR's base unit is the **yoctoNEAR** (10<sup>-24</sup> NEAR), which often involves counting lots of zeros.

`near-kit` solves this by letting you use human-readable strings. The library handles the conversion for you.

## Amounts

- **Use `"10.5 NEAR"` for token amounts.**
- **Use `"1 yocto"` for the smallest unit, often required for storage deposits.**

```typescript
// Send 10.5 NEAR
await near.transaction(sender).transfer(receiver, "10.5 NEAR").send()

// Call a function and attach exactly 1 yoctoNEAR
await near
  .transaction(sender)
  .functionCall(contract, "method", {}, { attachedDeposit: "1 yocto" })
  .send()
```

## Gas

- **Use `"30 Tgas"` for gas amounts.** (Tgas = Tera-gas = 10<sup>12</sup> gas units).

```typescript
await near
  .transaction(sender)
  .functionCall(
    contract,
    "method",
    {},
    {
      gas: "50 Tgas", // Attach 50 Tgas
    }
  )
  .send()
```

This approach makes your code more readable and less error-prone.

## Amount Helper Functions

For dynamic amounts, `near-kit` provides the `Amount` namespace with helper functions:

```typescript
import { Amount } from "near-kit"

// Create amounts programmatically
const deposit = Amount.NEAR(10) // "10 NEAR"
const fractional = Amount.NEAR(10.5) // "10.5 NEAR"
const tiny = Amount.yocto(1000000n) // "1000000 yocto"

// Use constants
const zero = Amount.ZERO // "0 yocto"
const one = Amount.ONE_NEAR // "1 NEAR"
const oneYocto = Amount.ONE_YOCTO // "1 yocto"

// Use in transactions
await near.transaction(sender).transfer(receiver, Amount.NEAR(amount)).send()
```

These helpers ensure type safety and prevent unit confusion when working with dynamic values.

## Formatting Balances

When displaying balances to users, use `formatAmount()` to convert yoctoNEAR to human-readable NEAR:

```typescript
import { formatAmount } from "near-kit"

// Get account balance (returns yoctoNEAR string)
const accountDetails = await near.getAccountDetails("alice.testnet")

// Format for display
const formatted = formatAmount(accountDetails.amount)
// "100.50 NEAR"

// Customize formatting
const customFormatted = formatAmount(accountDetails.amount, {
  precision: 4, // Show 4 decimal places (default: 2)
  trimZeros: true, // Remove trailing zeros
  includeSuffix: true, // Include " NEAR" suffix (default: true)
})
// "100.5 NEAR" (trailing zeros removed)

// Get just the number without suffix
const numberOnly = formatAmount(accountDetails.amount, {
  includeSuffix: false,
})
// "100.50"
```

**Note:** `near.getBalance()` already returns a formatted string, so you only need `formatAmount()` when working with raw yoctoNEAR values from other sources.
