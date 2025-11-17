# AI Agent Cookbook: Advanced Topics

This document covers advanced concepts that are crucial for building robust and efficient agents: Gas, storage costs, and creating complex, multi-action transactions.

## Gas Fees

On NEAR, computation and network bandwidth are paid for with gas. Every action in a transaction consumes gas.

### Human-Readable Gas Units

Instead of using raw gas units (e.g., `30000000000000`), `near-kit` allows you to use human-readable strings with `Tgas` (Tera-gas, or 10^12 gas units) as the unit. This is the standard unit for specifying gas limits.

- **`"30 Tgas"`** is the standard amount for a simple cross-contract call.
- **`"300 Tgas"`** is the maximum gas that can be attached to a single function call.

Gas is specified in the `options` object of a `.functionCall()` action.

```typescript
await near
  .transaction(sender)
  .functionCall(
    contract,
    "some_complex_method",
    {},
    {
      gas: "100 Tgas", // Attach 100 Tgas to this call
    }
  )
  .send();
```

If you do not specify a gas amount, a default value is used, which is usually sufficient for simple operations. For computationally intensive methods, you must attach enough gas.

## Storage Costs

Storing data on the NEAR blockchain is not free. Accounts must pay for the storage they use, a concept known as "storage staking". When an action causes a contract to store more data, the account that initiated the action must cover the cost by attaching a deposit.

### Attaching Deposits

Deposits are specified in NEAR tokens, and `near-kit` again provides a human-readable way to handle them.

- **`"1 NEAR"`**: Attaches 1 NEAR token.
- **`"0.01 NEAR"`**: Attaches 0.01 NEAR.
- **`"1 yocto"`**: Attaches the smallest possible unit of NEAR (10^-24 NEAR). This is often used to satisfy storage checks that require a deposit of at least 1 yoctoNEAR.

The deposit is specified in the `options` object of a `.functionCall()` action.

```typescript
// Call a method that requires a storage deposit
await near
  .transaction(sender)
  .functionCall(
    "nft-contract.testnet",
    "nft_mint",
    { token_id: "my-nft", metadata: { ... } },
    {
      // Attach 0.1 NEAR to cover the storage cost of the new NFT
      attachedDeposit: "0.1 NEAR",
    }
  )
  .send();
```

**Important**: If a contract call fails with a panic message like `"ERR_NOT_ENOUGH_DEPOSIT"`, it means the `attachedDeposit` was too low to cover the storage costs. The agent should retry the call with a larger deposit.

## Complex, Multi-Action Transactions

The `TransactionBuilder` allows for chaining multiple actions into a single, atomic transaction. This is powerful for setting up accounts, deploying contracts, and initializing them in one step. All actions in the chain either succeed together or fail together.

### Pattern: Create, Fund, Deploy, and Initialize

This pattern demonstrates a common workflow for creating a new contract account from scratch.

1.  **`createAccount`**: Create a new account.
2.  **`transfer`**: Send NEAR to the new account to cover its balance and future storage needs.
3.  **`addKey`**: Add an access key to the new account (optional, but good practice).
4.  **`deployContract`**: Deploy the Wasm bytecode to the new account.
5.  **`functionCall`**: Call an initialization method on the newly deployed contract.

```typescript
import { generateKey } from "near-kit";
import { readFileSync } from "fs";

const signerId = "your-app.testnet";
const newContractId = `my-new-contract.${signerId}`;
const contractWasm = readFileSync("./path/to/contract.wasm");
const newKey = generateKey();

const result = await near
  .transaction(signerId) // The app account pays for this setup
  .createAccount(newContractId)
  .transfer(newContractId, "5 NEAR") // Fund the new account
  .addKey(newKey.publicKey.toString(), { type: "fullAccess" }) // Add a new full access key
  .deployContract(newContractId, contractWasm) // Deploy the code
  .functionCall(
    newContractId,
    "init", // Call the initialization method
    { owner_id: signerId },
    { gas: "50 Tgas" }
  )
  .send();

console.log(`Contract ${newContractId} deployed and initialized successfully!`);
```

This entire chain is submitted as a single transaction. If any step fails (e.g., the `init` method panics), the entire setup is rolled back, and the `newContractId` account will not exist.
