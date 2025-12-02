# Action Reference

This page documents every method available on the `TransactionBuilder`.

To start a transaction:

```typescript
near.transaction(signerId: string)
```

## Token Operations

### `.transfer(receiverId, amount)`

Sends NEAR tokens from the signer to the receiver.

- **receiverId**: `string` - The account receiving the tokens.
- **amount**: `Amount` - The amount to send (e.g. `"10 NEAR"`, `"0.5 NEAR"`, `"1000 yocto"`).

### `.stake(publicKey, amount)`

Stakes NEAR with a validator.

- **publicKey**: `string` - The validator's public key.
- **amount**: `Amount` - The amount to stake.

## Contract Operations

### `.functionCall(contractId, methodName, args, options)`

Calls a method on a smart contract.

- **contractId**: `string` - The contract account ID.
- **methodName**: `string` - The method to call.
- **args**: `object | Uint8Array` - (Optional) Arguments. Objects are automatically JSON serialized.
- **options**: `object` - (Optional)
  - `gas`: `Gas` - Computation limit (default: `"30 Tgas"`).
  - `attachedDeposit`: `Amount` - NEAR to send to the contract (default: `"0 yocto"`).

### `.deployContract(accountId, code)`

Deploys Wasm code to an account. If the account already has a contract, it will be updated.

- **accountId**: `string` - The account to deploy to.
- **code**: `Uint8Array` - The raw bytes of the compiled Wasm file.

### `.stateInit(stateInit, options?)`

Deploys a contract to a deterministic account ID (NEP-616). The address is auto-derived from the initialization state. Idempotent—refunds if already deployed.

- **stateInit**: `object`
  - `code`: `{ accountId: string } | { codeHash: string }` - Global contract reference
  - `data`: `Map<string, string>` - (Optional) Initial storage
  - `deposit`: `Amount` - Storage cost reserve
- **options**: `{ refundTo?: string }` - (Optional) Custom refund recipient

```typescript
.stateInit({
  code: { accountId: "publisher.near" },
  data: new Map([["owner", "alice.near"]]),
  deposit: "1 NEAR",
})
```

See [Global Contracts](/in-depth/global-contracts.md#3-deterministic-account-ids-nep-616).

## Account Management

### `.createAccount(accountId)`

Creates a new account. This is often chained with `.transfer` (to fund it) and `.addKey` (to secure it).

- **accountId**: `string` - The full ID of the new account (must be a sub-account of the signer).

### `.deleteAccount(beneficiaryId)`

Deletes the signer's account and sends all remaining state rental funds to the `beneficiaryId`.

- **beneficiaryId**: `string` - The account that receives the remaining NEAR.

## Access Keys

### `.addKey(publicKey, permission)`

Adds a new access key to the account.

- **publicKey**: `string` - The public key to add (ed25519 or secp256k1).
- **permission**: `AccessKeyPermission` - One of:
  - `{ type: "fullAccess" }` - Can do anything.
  - `{ type: "functionCall", receiverId: string, methodNames?: string[], allowance?: Amount }` - Restricted to specific contracts/methods.

```typescript
// Full Access
.addKey(pk, { type: "fullAccess" })

// Restricted Access (e.g. for a frontend app)
.addKey(pk, {
  type: "functionCall",
  receiverId: "app.near",
  methodNames: ["vote", "comment"],
  allowance: "0.25 NEAR" // Gas allowance
})
```

### `.deleteKey(accountId, publicKey)`

Removes an access key.

- **accountId**: `string` - The account to remove the key from.
- **publicKey**: `string` - The public key to remove.

## Global Contracts

### `.publishContract(code, options?)`

Publishes contract code to the global registry so others can deploy it by reference.

- **code**: `Uint8Array` - The Wasm bytes.
- **options**: `object` - (Optional) Configuration for how the contract is identified:
  - `identifiedBy`: `"hash" | "account"` - How the contract is referenced:
    - `"account"` (default): Updatable by signer, identified by signer's account ID
    - `"hash"`: Immutable, identified by code hash

**Examples:**

```typescript
// Updatable contract (default) - identified by your account
.publishContract(wasm)

// Immutable contract - identified by hash
.publishContract(wasm, { identifiedBy: "hash" })
```

### `.deployFromPublished(reference)`

Deploys contract code that was previously published to the registry. This saves gas by avoiding uploading the full Wasm bytes.

- **reference**: One of:
  - `{ accountId: string }` - Account ID of the publisher (for updatable contracts)
  - `{ codeHash: string }` - Base58 hash of an immutable contract

## Advanced / Meta-Transactions

### `.delegate(options?)`

Instead of sending the transaction, this method signs it and returns a `SignedDelegateAction` payload. This is used for meta-transactions where a relayer pays the gas.

- **options**: `object` - (Optional)
  - `maxBlockHeight`: `bigint` - Expiration block.
  - `payloadFormat`: `"base64" | "bytes"` - Output format.

Returns: `{ signedDelegateAction, payload, format }`.

### `.signedDelegateAction(signedDelegate)`

Adds a pre-signed delegate action to this transaction. Used by relayers to submit a user's action.

### `.signWith(key)`

Overrides the signer for _this specific transaction_. Does not change the global `Near` configuration.

- **key**: `string | Signer` - A private key string OR a custom signer function.

```typescript
// Use a specific key just for this transaction
.signWith("ed25519:...")

// Use a custom signer (e.g. hardware wallet)
.signWith(async (hash) => {
  return ledger.sign(hash);
})
```
