# Building Transactions

The `near.transaction()` builder is the heart of `near-kit`. It provides a fluent, chainable API to construct and send transactions with one or more actions.

While the [simple write methods](../core-concepts/writing-data.md) are convenient for single operations, the transaction builder is essential when you need to perform multiple operations in a single, atomic batch **to a single receiver account**.

## The Fluent API

You start a transaction by specifying the **signer account ID**—the account that will sign the transaction and pay for the [gas fees](../core-concepts/units.md).

```ts
near.transaction("signer.near") // 'signer.near' pays for this transaction
```

From there, you can chain one or more actions. The transaction is not sent until you call `.send()`.

### Atomic by Nature (Within a Transaction)

All actions added to a single transaction are executed **sequentially as one atomic unit on-chain**:

- If **all actions succeed**, the state changes produced by those actions are committed and any promises (cross-contract calls) they created are emitted.
- If **any action fails** (e.g. a function call panics, a transfer hits insufficient balance, etc.), the entire transaction is reverted:

  - none of the state changes from _any_ action are applied, and
  - any promises created by earlier actions in that transaction are discarded and never sent.

In other words: **within one transaction / receipt, it’s all-or-nothing.**

```admonish warning title="This atomicity boundary stops at the transaction itself" collapsible=false
Cross-contract calls created _by_ your transaction run later as separate receipts. Once those child receipts have been emitted and executed on other contracts, their effects are **not** automatically rolled back if something else fails later—you must implement any “compensating logic” yourself in callbacks.
```

## Chaining Multiple Actions

The real power of the builder comes from combining actions. This example creates a new sub-account, transfers funds to it, adds a full access key, and deploys a contract to it, all in a single atomic transaction.

```ts
import { generateKey } from "near-kit"
import { readFileSync } from "fs"

const newAccount = `sub.${YOUR_ACCOUNT_ID}`
const newKey = generateKey()
const contractWasm = readFileSync("./path/to/contract.wasm")

const result = await near
  .transaction(YOUR_ACCOUNT_ID)
  .createAccount(newAccount)
  .transfer(newAccount, "5 NEAR")
  .addKey(newKey.publicKey.toString(), { type: "fullAccess" })
  .deployContract(newAccount, contractWasm)
  .send()

console.log(`${newAccount} created and configured successfully!`)
```

## Available Actions

The builder supports a wide range of actions, from simple transfers to advanced features like staking and deploying from a global registry.

➡️ **See the [Available Actions](../reference/actions.md) guide for a complete list and detailed examples of every action.**

## Sending the Transaction

Once you have added all your actions, you send the transaction using the `.send()` method.

```ts
const result = await near
  .transaction("signer.near")
  .transfer("receiver.near", "1 NEAR")
  .send()
```

The `.send()` method signs the transaction and broadcasts it to the network. By default, it waits for the transaction to be executed and returns a detailed result.

### Controlling the Response with `waitUntil`

You can control how long the client waits for a response and how much detail you get back by using the `waitUntil` option in the `.send()` method. This is useful for balancing between speed and finality guarantees.

```ts
// Wait for full finality before returning
const finalResult = await near
  .transaction("signer.near")
  .functionCall("contract.near", "some_method", {})
  .send({ waitUntil: "FINAL" })
```

```admonish info title="Understanding Finality Options"

When you submit a transaction, it first gets validated for structure (if it can be deserialized from borsh). If not, it will be rejected by the RPC. Otherwise:

- **NONE** (Fire and Forget)

  - **Returns:** Immediately after the transaction is validated.
  - **Details:** The transaction hasn’t even started executing, but in most cases, it will start in the next block. The RPC returns a response confirming the transaction is fine.

- **INCLUDED**

  - **Returns:** When the transaction hits the validator node.
  - **Details:** It gets validated (signature matches, account charged gas pre-payment, access key nonce updated), included in the chunk/block, and converted into a receipt for execution. The RPC returns a response, but the transaction has just started execution, so no return value or logs are available yet.

- **EXECUTED_OPTIMISTIC** (Default)

  - **Returns:** Once the receipt chain finishes execution.
  - **Details:** In the next blocks, the receipt will get executed (which may produce more receipts for cross-account interaction). Once this chain finishes, the RPC returns the response.

- **INCLUDED_FINAL**

  - **Returns:** When the block that got the transaction included is finalized.
  - **Details:** This can resolve before or after `EXECUTED_OPTIMISTIC`, depending on how long the receipts execution takes.

- **EXECUTED**

  - **Returns:** When both `INCLUDED_FINAL` and `EXECUTED_OPTIMISTIC` conditions are met.

- **FINAL**
  - **Returns:** When the block that has the last non-refund receipt is finalized.

```

#### Inspecting the Result

When you use the default `waitUntil` or `"FINAL"`, you get a rich result object. Here are the most important properties:

- `result.transaction.hash`: The unique hash of your transaction.
- `result.status.SuccessValue`: If a contract method returned a value, it will be here as a base64-encoded string. You need to decode and parse it.
- `result.status.Failure`: If the transaction failed at the protocol level, details will be here.
- `result.receipts_outcome[...].outcome.logs`: An array of log messages from the contract, essential for debugging.
- `result.receipts_outcome[...].outcome.status.Failure`: If a specific action (like a cross-contract call) failed, the `FunctionCallError` details will be here.
