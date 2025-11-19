# Global Contracts

On NEAR, typically every smart contract account holds its own copy of the Wasm bytecode. This means if you deploy the same Token contract to 1,000 accounts, you are paying for that storage 1,000 times.

**Global Contracts** allow you to publish the Wasm bytecode **once** to a global registry. You can then deploy instances of that contract by simply referencing its hash.

## Use Cases

- **Factories:** If you are building a "DAO Factory" or "Token Factory" that deploys new contracts for users, this reduces the deployment cost from ~4 NEAR (storage) to <0.1 NEAR.
- **Standard Libraries:** Publish a standard implementation (like a multisig) that everyone trusts and uses.

## 1. Publishing Code

Publishing requires a storage deposit proportional to the size of your Wasm file (approx 1 NEAR per 100KB).

### Immutable Contracts (Trustless)

If you publish without a specific publisher ID, the code is indexed by its **SHA-256 hash**. It can never be changed. This is ideal for trustless protocols.

```typescript
import { readFileSync } from "fs"
import { createHash } from "crypto" // Standard Node.js module

const wasm = readFileSync("./my-token.wasm")

// 1. Calculate the hash locally
// near-kit accepts the raw Buffer/Uint8Array
const codeHash = createHash("sha256").update(wasm).digest()

// 2. Publish the code
await near.transaction("deployer.near").publishContract(wasm).send()

console.log("Contract published!")
```

### Mutable Contracts (Updatable)

If you associate the code with an `accountId`, you can update the code later by running `publishContract` again. Users who deploy referencing your Account ID will always get the _latest_ version you published.

```typescript
await near
  .transaction("factory.near")
  .publishContract(wasm, "factory.near") // Associate with this account
  .send()
```

## 2. Deploying by Reference

To deploy, use `deployFromPublished` instead of `deployContract`. This action is extremely cheap because it uses almost no bandwidth or new storage.

### Deploying from Hash (Immutable)

```typescript
const codeHash = "5FzD8..." // The hash from Step 1

await near
  .transaction("user.near")
  .createAccount("token.user.near")
  .transfer("token.user.near", "1 NEAR") // Storage for state, not code!
  .deployFromPublished({ codeHash })
  .functionCall("token.user.near", "init", { supply: "1000" })
  .send()
```

### Deploying from Account (Mutable)

This deploys whatever code `factory.near` has currently published.

```typescript
await near
  .transaction("user.near")
  .createAccount("dao.user.near")
  .deployFromPublished({ accountId: "factory.near" })
  .send()
```
