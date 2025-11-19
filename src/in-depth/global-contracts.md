# Global Contracts

On NEAR, typically every smart contract account holds its own copy of the Wasm bytecode. This means if you deploy the same Token contract to 1,000 accounts, you are paying for that storage 1,000 times.

**Global Contracts** allow you to publish the Wasm bytecode **once** to a global registry. You can then deploy instances of that contract by simply referencing its hash.

## Use Cases

- **Factories:** If you are building a "DAO Factory" or "Token Factory" that deploys new contracts for users, this reduces the deployment cost from ~4 NEAR (storage) to <0.1 NEAR.
- **Standard Libraries:** Publish a standard implementation (like a multisig) that everyone trusts and uses.

## 1. Publishing Code

Publishing requires a storage deposit proportional to the size of your Wasm file (approx 1 NEAR per 100KB).

### Updatable Contracts (Default)

By default, published contracts are identified by **your account ID**. This allows you to update the code later by publishing again. All users who deployed referencing your account will automatically use the latest version you published.

```typescript
import { readFileSync } from "fs"

const wasm = readFileSync("./my-token.wasm")

// Publish updatable contract (identified by your account)
await near.transaction("factory.near").publishContract(wasm).send()

// Later, you can update it:
const updatedWasm = readFileSync("./my-token-v2.wasm")
await near
  .transaction("factory.near")
  .publishContract(updatedWasm) // Overwrites previous version
  .send()
```

Users deploying with `{ accountId: "factory.near" }` will always get your latest published version.

### Immutable Contracts (Trustless)

For trustless protocols or when you want guaranteed immutability, publish the code identified by its **SHA-256 hash**. It can never be changed.

```typescript
import { readFileSync } from "fs"
import { createHash } from "crypto" // Standard Node.js module

const wasm = readFileSync("./my-token.wasm")

// 1. Calculate the hash locally
const codeHash = createHash("sha256").update(wasm).digest()

// 2. Publish as immutable (identified by hash)
await near
  .transaction("deployer.near")
  .publishContract(wasm, { identifiedBy: "hash" })
  .send()

console.log("Immutable contract published!")
console.log("Hash:", codeHash.toString("base64"))
```

## 2. Deploying by Reference

To deploy, use `deployFromPublished` instead of `deployContract`. This action is extremely cheap because it uses almost no bandwidth or new storage.

### Deploying from Account (Updatable)

This deploys whatever code `factory.near` has currently published. If they update the contract later, you'll automatically get the latest version.

```typescript
await near
  .transaction("user.near")
  .createAccount("dao.user.near")
  .transfer("dao.user.near", "1 NEAR") // Storage for state, not code!
  .deployFromPublished({ accountId: "factory.near" })
  .functionCall("dao.user.near", "init", {})
  .send()
```

### Deploying from Hash (Immutable)

For contracts published with `identifiedBy: "hash"`, reference them by their SHA-256 hash:

```typescript
const codeHash = "5FzD8..." // Base58-encoded hash from publishing

await near
  .transaction("user.near")
  .createAccount("token.user.near")
  .transfer("token.user.near", "1 NEAR")
  .deployFromPublished({ codeHash })
  .functionCall("token.user.near", "init", { supply: "1000" })
  .send()
```
