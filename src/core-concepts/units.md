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
  .functionCall(
    contract,
    "method",
    {},
    {
      attachedDeposit: "1 yocto",
    }
  )
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