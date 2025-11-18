# AI Agent Cookbook: Transaction Builder

This document provides patterns for creating and sending transactions using the `TransactionBuilder`. This is the primary mechanism for performing state-changing operations on the NEAR blockchain.

## Core Concepts

-   Start a transaction with `near.transaction(signerId)`.
-   Chain one or more actions (`.transfer()`, `.functionCall()`, etc.).
-   All actions in a single chain are **atomic** (they all succeed or all fail).
-   Send the transaction to the network with `.send()`.

```admonish warning title="Atomicity Boundary"
The atomicity guarantee applies only to the actions within the transaction itself. It does **not** extend to cross-contract calls, which are executed as separate receipts in subsequent blocks. If a cross-contract call fails, it will not cause the original transaction to be reverted.
```

## Type Definition: `Action`

An action is a single operation to be performed in a transaction. The most common ones are represented by the following types:

```typescript
// A transfer of NEAR tokens
type TransferAction = {
  transfer: { deposit: bigint };
};

// A call to a smart contract method
type FunctionCallAction = {
  functionCall: {
    methodName: string;
    args: Uint8Array;
    gas: bigint;
    deposit: bigint;
  };
};

// Creation of a new account
type CreateAccountAction = {
  createAccount: {};
};

// Deployment of a smart contract
type DeployContractAction = {
  deployContract: { code: Uint8Array };
};

// For all other actions, refer to the library's source code or official NEAR documentation.
```

---

## Patterns

### Pattern 1: Send NEAR

Use the `.transfer()` action.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet", privateKey: "...", defaultSignerId: "alice.testnet" });

// Send 1.5 NEAR from 'alice.testnet' to 'bob.testnet'
const result = await near
  .transaction("alice.testnet")
  .transfer("bob.testnet", "1.5 NEAR")
  .send();
```

### Pattern 2: Call a Contract Method

Use the `.functionCall()` action.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet", privateKey: "...", defaultSignerId: "alice.testnet" });

// Call 'add_message' on a guestbook contract with an attached deposit
const result = await near
  .transaction("alice.testnet")
  .functionCall(
    "guestbook.testnet",
    "add_message",
    { text: "Hello from near-kit!" },
    { gas: "30 Tgas", attachedDeposit: "0.1 NEAR" }
  )
  .send();
```

### Pattern 3: Create an Account

Use the `.createAccount()` action.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet", privateKey: "...", defaultSignerId: "alice.testnet" });

// Create the account 'new-account.testnet'
const result = await near
  .transaction("alice.testnet")
  .createAccount("new-account.testnet")
  .send();
```

### Pattern 4: Deploy a Contract

Use the `.deployContract()` action. This is often chained with `.createAccount()`.

```typescript
import { Near } from "near-kit";
import { readFileSync } from "fs";
const near = new Near({ network: "testnet", privateKey: "...", defaultSignerId: "alice.testnet" });

const contractWasm = readFileSync("./path/to/contract.wasm");

// Create an account and deploy a contract to it in one transaction
const result = await near
  .transaction("alice.testnet")
  .createAccount("my-contract.testnet")
  .deployContract("my-contract.testnet", contractWasm)
  .send();
```

### Pattern 5: Add an Access Key

Use the `.addKey()` action to add a key with specific permissions to an account.

```typescript
import { Near, generateKey } from "near-kit";
const near = new Near({ network: "testnet", privateKey: "...", defaultSignerId: "alice.testnet" });

const newKey = generateKey(); // Generates a new ed25519 key pair

// Add a Function Call access key to 'my-app.testnet'
const result = await near
  .transaction("my-app.testnet")
  .addKey(newKey.publicKey.toString(), {
    type: "functionCall",
    receiverId: "some-contract.testnet",
    methodNames: ["some_method"],
    allowance: "0.25 NEAR",
  })
  .send();
```

### Pattern 6: Delete an Access Key

Use the `.deleteKey()` action.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet", privateKey: "...", defaultSignerId: "alice.testnet" });

// The public key to delete, in 'ed25519:<bs58_string>' format
const publicKeyToDelete = "ed25519:...";

// Delete a specific access key from 'alice.testnet'
const result = await near
  .transaction("alice.testnet")
  .deleteKey("alice.testnet", publicKeyToDelete)
  .send();
```

### Pattern 7: Delete an Account

Use the `.deleteAccount()` action. The remaining balance will be sent to the beneficiary.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet", privateKey: "...", defaultSignerId: "alice.testnet" });

// Delete 'alice.testnet' and send remaining funds to 'bob.testnet'
const result = await near
  .transaction("alice.testnet")
  .deleteAccount("bob.testnet")
  .send();
```

### Pattern 8: Stake NEAR with a Validator

Use the `.stake()` action to stake tokens and earn rewards.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet", privateKey: "...", defaultSignerId: "alice.testnet" });

// The public key of the validator to stake with
const validatorKey = "ed25519:..."; 
const amountToStake = "1000 NEAR";

// Stake 1000 NEAR from 'alice.testnet' with the specified validator
const result = await near
  .transaction("alice.testnet")
  .stake(validatorKey, amountToStake)
  .send();
```

### Pattern 9: Publish a Contract to the Global Registry

Use `.publishContract()` to make a contract's code available for others to deploy by reference.

```typescript
import { Near } from "near-kit";
import { readFileSync } from "fs";
const near = new Near({ network: "testnet", privateKey: "...", defaultSignerId: "publisher.testnet" });

const contractWasm = readFileSync("./path/to/contract.wasm");

// Publish an immutable contract, identified by its code hash
const result = await near
  .transaction("publisher.testnet")
  .publishContract(contractWasm)
  .send();
```

### Pattern 10: Deploy a Contract from the Global Registry

Use `.deployFromPublished()` to create a new contract instance from a published code reference.

```typescript
import { Near } from "near-kit";
const near = new Near({ network: "testnet", privateKey: "...", defaultSignerId: "deployer.testnet" });

// Reference the published contract by its code hash
const codeHash = "5FzD8..."; // The hash of the contract code from the publish step

// Deploy a new instance from the published code
const result = await near
  .transaction("deployer.testnet")
  .createAccount("new-instance.testnet")
  .deployFromPublished({ codeHash })
  .send();
```

### Pattern 11: Create and Submit a Delegate Action (for Meta-Transactions)

Use `.delegate()` instead of `.send()` to sign an action off-chain. The result includes both the structured action and an encoded payload that can be forwarded to a relayer.

```typescript
import { Near } from "near-kit";

// User's client
const userNear = new Near({ network: "testnet", privateKey: "...", defaultSignerId: "user.testnet" });

// User signs a function call off-chain (no gas cost)
const { signedDelegateAction, payload, format } = await userNear
  .transaction("user.testnet")
  .functionCall("guestbook.testnet", "add_message", { text: "Hello from meta-tx!" })
  .delegate();
// `payload` (base64 by default) can now be sent to a relayer service.
// `signedDelegateAction` remains available locally for wallet UI/validation.
// Need raw bytes? Pass { payloadFormat: "bytes" } to .delegate().
```

### Pattern 12: Submit a Signed Delegate Action (as a Relayer)

A relayer uses `.signedDelegateAction()` to submit the user's action to the network, paying the gas on their behalf.

```typescript
import { Near, decodeSignedDelegateAction } from "near-kit";
// Assume `payload` (base64 string) was received from a user.

// Relayer's client
const relayerNear = new Near({ network: "testnet", privateKey: "...", defaultSignerId: "relayer.testnet" });

const signedDelegate = decodeSignedDelegateAction(payload);

// Relayer wraps the user's action and sends it
const result = await relayerNear
  .transaction("relayer.testnet")
  .signedDelegateAction(signedDelegate)
  .send();
```
