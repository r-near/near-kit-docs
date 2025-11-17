# AI Agent Cookbook: Sandbox Testing

This document provides a pattern for using the `near-kit` Sandbox for automated testing. The Sandbox provides a local, isolated NEAR blockchain, which is essential for reliable and fast testing.

## Core Concept

The Sandbox is managed by the `Sandbox` class. The typical testing flow is:
1.  **Start Sandbox**: Before tests run, start a new Sandbox instance.
2.  **Run Tests**: Execute tests against the local blockchain.
3.  **Stop Sandbox**: After tests complete, shut down the Sandbox to clean up resources.

---

## Pattern: Sandbox Usage in a Test File

This pattern shows how to use the Sandbox within a test file using a testing framework like `bun:test`, `jest`, or `vitest`.

```typescript
import { Near } from "near-kit";
import { Sandbox } from "near-kit/sandbox";
import { beforeAll, afterAll, test, expect } from "bun:test"; // Or your test framework

// Define a scope for the sandbox instance
let sandbox: Sandbox;
let near: Near;

// Use a describe block to group tests that use the same sandbox
describe("Smart Contract Tests", () => {
  // 1. Start the Sandbox before all tests in this block
  beforeAll(async () => {
    sandbox = await Sandbox.start();
    
    // Initialize near-kit to connect to the new sandbox instance
    near = new Near({ network: sandbox });
    
    console.log(`Sandbox started with RPC at ${sandbox.rpcUrl}`);
  }, 120000); // Use a long timeout for the first run, as it may download binaries

  // 2. Stop the Sandbox after all tests in this block have finished
  afterAll(async () => {
    if (sandbox) {
      await sandbox.stop();
      console.log("Sandbox stopped.");
    }
  });

  // 3. Write your tests
  test("should create an account and transfer funds", async () => {
    // The sandbox provides a pre-funded root account
    const rootAccount = sandbox.rootAccount;
    const newAccountId = `test-account.${rootAccount.id}`;

    // The root account's key is automatically loaded into the Near instance
    const result = await near
      .transaction(rootAccount.id)
      .createAccount(newAccountId)
      .transfer(newAccountId, "100 NEAR")
      .send();

    // Assert that the transaction was successful
    expect(result.final_execution_status).toContain("EXECUTED");

    // Verify the outcome
    const balance = await near.getBalance(newAccountId);
    expect(balance).toBe("100.00");
  });

  test("should deploy a contract and call a method", async () => {
    const rootAccount = sandbox.rootAccount;
    const contractId = `my-contract.${rootAccount.id}`;
    
    // In a real scenario, you would load your contract's Wasm file
    // const contractWasm = readFileSync("./path/to/contract.wasm");

    // For this example, we'll use a placeholder wasm
    const placeholderWasm = Buffer.from([
      0x00, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00,
    ]);

    // Deploy the contract
    await near
      .transaction(rootAccount.id)
      .createAccount(contractId)
      .deployContract(contractId, placeholderWasm)
      .send();
      
    // You can now interact with the deployed contract
    // const response = await near.view(contractId, "some_view_method", {});
    // expect(response).toBe("some_value");
  });
});
```

### The `sandbox` Object

The `Sandbox.start()` method returns a `Sandbox` instance with the following key properties:

-   `sandbox.rpcUrl`: The RPC endpoint for this sandbox instance (e.g., `http://127.0.0.1:42693`).
-   `sandbox.networkId`: The network ID, which is always `'localnet'`.
-   `sandbox.rootAccount.id`: The ID of the pre-funded root account (e.g., `test.near`).
-   `sandbox.rootAccount.secretKey`: The private key for the root account.

When you pass the `sandbox` object directly to `new Near({ network: sandbox })`, `near-kit` automatically configures the RPC connection and loads the root account's key into an in-memory keystore, making it ready to sign transactions immediately.
