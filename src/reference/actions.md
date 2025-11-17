# Available Actions

The transaction builder supports a wide range of actions that can be chained together to create powerful, atomic transactions.

```admonish warning title="Understanding Atomicity Boundaries"
All actions within a single transaction are **atomic**—they either all succeed or all fail together. If any action panics or fails, the entire transaction is reverted.

However, this guarantee does **not** extend to cross-contract calls, which execute as separate receipts in later blocks. If a child receipt fails, it will not roll back the parent transaction. For more details, see the guide on [Building Transactions](../transactions/builder.md#atomic-by-nature-within-a-transaction).
```

## `transfer()`

Sends NEAR tokens from the signer to a receiver.

-   `receiverId: string`: The account to receive the tokens.
-   `amount: Amount`: The amount of NEAR to send (e.g., `"10.5 NEAR"`).

```typescript
await near
  .transaction("alice.near")
  .transfer("bob.near", "10.5 NEAR")
  .send();
```

## `functionCall()`

Calls a method on a smart contract.

-   `contractId: string`: The contract to call.
-   `methodName: string`: The name of the method to call.
-   `args: object | Uint8Array`: The arguments to pass to the method.
-   `options?: { gas?: Gas, attachedDeposit?: Amount }`: Optional gas and deposit.

```typescript
await near
  .transaction("alice.near")
  .functionCall(
    "market.near",
    "buy",
    { nft_id: "token1" },
    { gas: "50 Tgas", attachedDeposit: "10 NEAR" }
  )
  .send();
```

## `createAccount()`

Creates a new account. The ID of the new account becomes the `receiverId` for any subsequent actions in the same transaction.

-   `accountId: string`: The ID for the new account.

```typescript
await near
  .transaction("alice.near")
  .createAccount("new-account.near")
  .transfer("new-account.near", "5 NEAR") // Transfer to the newly created account
  .send();
```

## `deployContract()`

Deploys a Wasm smart contract to an account.

-   `accountId: string`: The account to deploy the contract to.
-   `code: Uint8Array`: The Wasm bytecode of the contract.

```typescript
import { readFileSync } from "fs";
const contractWasm = readFileSync("./my-contract.wasm");

await near
  .transaction("contract-account.near")
  .deployContract("contract-account.near", contractWasm)
  .send();
```

## `addKey()`

Adds a new access key to an account.

-   `publicKey: string`: The public key to add.
-   `permission: AccessKeyPermission`: The permissions for the key (`"fullAccess"` or `"functionCall"`).

```typescript
import { generateKey } from "near-kit";

const newKey = generateKey();

// Add a full access key
await near
  .transaction("alice.near")
  .addKey(newKey.publicKey.toString(), { type: "fullAccess" })
  .send();

// Add a key restricted to calling specific methods
await near
  .transaction("alice.near")
  .addKey(newKey.publicKey.toString(), {
    type: "functionCall",
    receiverId: "some-contract.near",
    methodNames: ["method1", "method2"],
    allowance: "0.25 NEAR",
  })
  .send();
```

## `deleteKey()`

Deletes an access key from an account.

-   `accountId: string`: The account from which to delete the key.
-   `publicKey: string`: The public key to delete.

```typescript
await near
  .transaction("alice.near")
  .deleteKey("alice.near", "ed25519:...")
  .send();
```

## `deleteAccount()`

Deletes an account and transfers its remaining balance to a beneficiary.

-   `beneficiaryId: string`: The account that will receive the remaining funds.

```typescript
// alice.near will be deleted, and its balance sent to bob.near
await near
  .transaction("alice.near")
  .deleteAccount("bob.near")
  .send();
```

## `stake()`

Stakes NEAR with a validator to earn rewards.

-   `publicKey: string`: The public key of the validator.
-   `amount: Amount`: The amount of NEAR to stake.

```typescript
await near
  .transaction("alice.near")
  .stake("ed25519:validatorPublicKey...", "1000 NEAR")
  .send();
```

## `publishContract()`

Publishes a contract to the global registry, allowing other accounts to deploy it by reference.

- `code: Uint8Array`: The compiled contract code bytes.
- `publisherId?: string`: Optional account ID to make the contract mutable (updatable).

```typescript
// Publish an immutable contract (identified by its code hash)
await near.transaction("alice.near")
  .publishContract(contractCode)
  .send();

// Publish a mutable contract (identified by the publisher's account ID)
await near.transaction("alice.near")
  .publishContract(contractCode, "alice.near")
  .send();
```

## `deployFromPublished()`

Deploys a contract that was previously published to the global registry.

- `reference: { codeHash: string | Uint8Array } | { accountId: string }`: A reference to the published contract.

```typescript
// Deploy from an immutable contract's code hash
await near.transaction("my-new-contract.near")
  .deployFromPublished({ codeHash: "5FzD8..." })
  .send();

// Deploy from a mutable contract's publisher
await near.transaction("my-other-contract.near")
  .deployFromPublished({ accountId: "contract-publisher.near" })
  .send();
```