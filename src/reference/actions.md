# Action Reference

These methods are available on the `TransactionBuilder`.

## Essentials

### `.transfer(receiverId, amount)`

Sends NEAR tokens.

- `amount`: String (e.g., `"10 NEAR"`, `"0.5 NEAR"`).

### `.functionCall(contractId, method, args, options)`

Calls a smart contract.

- `args`: Object or Uint8Array.
- `options`: `{ gas?: string, attachedDeposit?: string }`.

### `.createAccount(accountId)`

Creates a new sub-account. The `accountId` must be a sub-account of the signer (e.g., `sub.signer.near`).

### `.deleteAccount(beneficiaryId)`

Deletes the signer's account and sends all remaining NEAR to the `beneficiaryId`.

## Keys

### `.addKey(publicKey, permission)`

Adds an access key to the account.

- `permission`: `{ type: "fullAccess" }` or `{ type: "functionCall", ... }`.

### `.deleteKey(publicKey)`

Removes an access key.

## Advanced

### `.deployContract(accountId, code)`

Updates or deploys the Wasm code for the account.

- `code`: Uint8Array (file bytes).

### `.stake(publicKey, amount)`

Stakes NEAR with a validator.

### `.publishContract(code)`

Publishes code to the global registry.

### `.deployFromPublished(reference)`

Deploys code from the global registry.
