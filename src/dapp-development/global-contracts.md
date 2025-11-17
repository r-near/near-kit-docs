# Global Contracts

The Global Contracts feature allows developers to publish a smart contract's Wasm bytecode to a global, on-chain registry. Once published, any account can deploy an instance of that contract by referencing the published code, rather than having to include the full bytecode in their own deployment transaction.

This has several benefits:
- **Reduced Gas Costs**: Deploying a contract by reference is significantly cheaper than deploying with the full bytecode.
- **Code Re-use**: Standard contracts (like fungible tokens or NFTs) can be published once and re-used by many different applications.
- **Discoverability**: It creates an on-chain registry of standard, open-source contracts.

`near-kit` provides two new transaction actions to support this feature: `publishContract` and `deployFromPublished`.

## Publishing a Contract

You can publish a contract's code to the global registry using the `publishContract` action.

- `code: Uint8Array`: The Wasm bytecode of the contract.
- `publisherId?: string`: An optional account ID. If provided, the contract is **mutable**, meaning the publisher can update the code later. If omitted, the contract is **immutable** and is identified forever by the hash of its code.

### Example: Publishing an Immutable Contract

An immutable contract is identified by the SHA-256 hash of its bytecode. This guarantees that any contract deployed from this reference will always have the exact same code.

```typescript
import { readFileSync } from "fs";
const contractCode = readFileSync("./my-ft-contract.wasm");

// The signer ('publisher.near') pays to publish the code.
// The code is now available for anyone to use via its hash.
await near.transaction("publisher.near")
  .publishContract(contractCode)
  .send();
```

### Example: Publishing a Mutable Contract

A mutable contract is identified by the account ID of its publisher. This allows the publisher to upload new versions of the code, which can be useful for bug fixes or upgrades. Accounts that deploy from this reference will always get the latest version published by that account.

```typescript
import { readFileSync } from "fs";
const contractCode = readFileSync("./my-nft-contract.wasm");

// The contract is published and associated with 'publisher.near'.
// 'publisher.near' can update this code in the future.
await near.transaction("publisher.near")
  .publishContract(contractCode, "publisher.near")
  .send();
```

## Deploying from a Published Contract

Once a contract is in the global registry, any account can deploy it using the `deployFromPublished` action. This action is used in place of `deployContract`.

- `reference`: An object specifying how to find the published contract. It can be either:
    - `{ codeHash: string | Uint8Array }`: For immutable contracts.
    - `{ accountId: string }`: For mutable contracts.

### Example: Deploying from a Code Hash

This creates a new contract instance on `my-ft-instance.near` using the code from the immutable contract we published earlier.

```typescript
// The codeHash would be known from the output of the publish transaction
// or by hashing the original Wasm file.
const codeHash = "5FzD8..."; // Base58-encoded SHA-256 hash of the contract code

// 'deployer.near' creates a new contract instance.
// This transaction is much cheaper because it doesn't include the Wasm bytes.
await near.transaction("deployer.near")
  .createAccount("my-ft-instance.near")
  .deployFromPublished({ codeHash })
  .functionCall("my-ft-instance.near", "init", { owner: "deployer.near" })
  .send();
```

### Example: Deploying from a Publisher Account ID

This creates a new instance using the latest code published by `publisher.near`.

```typescript
// 'deployer.near' creates a new contract instance from the mutable source.
await near.transaction("deployer.near")
  .createAccount("my-nft-instance.near")
  .deployFromPublished({ accountId: "publisher.near" })
  .functionCall("my-nft-instance.near", "init", { owner: "deployer.near" })
  .send();
```