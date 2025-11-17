# AI Agent Cookbook: End-to-End Example

This document provides a complete, end-to-end example of an agent script that uses `near-kit` to perform a series of on-chain actions. This script demonstrates how to tie together initialization, transaction building, sandbox testing, and error handling.

## Scenario

The agent will perform the following tasks in a local sandbox environment:
1.  Start a new sandbox instance.
2.  Create a new account for a contract.
3.  Deploy a simple "status message" smart contract to the new account.
4.  Call the contract to set an initial status message.
5.  View the contract to verify the status message was set correctly.
6.  Clean up by stopping the sandbox.

This example uses `bun:test` for structure, but the logic can be adapted to any Node.js or Bun script.

## The Smart Contract

For this example, we'll use a simple pre-compiled "status message" contract. You would typically compile your own contract to Wasm, but for simplicity, we'll use a placeholder.

## End-to-End Agent Script

```typescript
import { Near, FunctionCallError } from "near-kit";
import { Sandbox } from "near-kit/sandbox";
import { afterAll, beforeAll, expect, test } from "bun:test";
import { readFileSync } from "node:fs";

// A simple status message contract (replace with your actual contract Wasm)
// This placeholder is a minimal valid Wasm module.
const STATUS_CONTRACT_WASM = Buffer.from([
  0x00, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00, 0x01, 0x04, 0x01, 0x60,
  0x00, 0x00, 0x03, 0x02, 0x01, 0x00, 0x07, 0x08, 0x01, 0x04, 0x6d, 0x61,
  0x69, 0x6e, 0x00, 0x00, 0x0a, 0x04, 0x01, 0x02, 0x00, 0x0b,
]);

describe("E2E Agent Workflow", () => {
  let sandbox: Sandbox;
  let near: Near;

  beforeAll(async () => {
    try {
      console.log("Starting sandbox...");
      sandbox = await Sandbox.start();
      near = new Near({ network: sandbox });
      console.log("Sandbox started successfully.");
    } catch (error) {
      console.error("Failed to start sandbox:", error);
      throw error; // Fail the test suite if sandbox doesn't start
    }
  }, 120000); // Long timeout for potential binary download

  afterAll(async () => {
    if (sandbox) {
      await sandbox.stop();
      console.log("Sandbox stopped.");
    }
  });

  test("should deploy a guestbook contract, add a message, and verify it", async () => {
    const rootAccount = sandbox.rootAccount;
    const contractId = `guestbook-contract.${rootAccount.id}`;
    const message = "Hello from the agent!";

    // --- 1. Load Contract Wasm ---
    // In a real agent, this Wasm might be fetched from a URL or provided as input.
    // For this example, we load it from the project's test files.
    const wasmPath = "../../../tests/contracts/guestbook.wasm";
    const wasm = readFileSync(wasmPath);
    console.log("Loaded guestbook.wasm");

    // --- 2. Create Account & Deploy Contract ---
    console.log(`Deploying contract to ${contractId}...`);
    try {
      const deployResult = await near
        .transaction(rootAccount.id)
        .createAccount(contractId)
        .transfer(contractId, "5 NEAR") // Add some balance for storage
        .deployContract(contractId, wasm)
        .send({ waitUntil: "FINAL" });

      expect(deployResult.final_execution_status).toBe("FINAL");
      console.log("Contract deployed successfully.");
    } catch (error) {
      console.error("Failed to deploy contract:", error);
      throw error;
    }

    // --- 3. Call the Contract to Add a Message ---
    console.log(`Adding message: "${message}"`);
    try {
      await near
        .transaction(rootAccount.id)
        .functionCall(
          contractId,
          "add_message",
          { text: message },
          { attachedDeposit: "0.1 NEAR" } // Guestbook requires a deposit
        )
        .send({ waitUntil: "FINAL" });
      console.log("Message added successfully.");
    } catch (error) {
      console.error("Failed to add message:", error);
      throw error;
    }

    // --- 4. View the Contract to Verify State ---
    console.log("Verifying message on-chain...");
    try {
      const messages = await near.view<Array<{ sender: string; text: string }>>(
        contractId,
        "get_messages",
        {}
      );
      
      const lastMessage = messages.pop();
      expect(lastMessage?.text).toBe(message);
      expect(lastMessage?.sender).toBe(rootAccount.id);
      console.log(`Verified message: "${lastMessage?.text}" from ${lastMessage?.sender}`);
    } catch (error) {
      console.error("Failed to verify message:", error);
      throw error;
    }
  });
});
```

### How an Agent Would Use This

-   **Automation**: This entire script can be run automatically on a schedule or triggered by an event.
-   **Error Handling**: The `try...catch` blocks are crucial. An intelligent agent would parse the specific error (`FunctionCallError`, `InvalidTransactionError`, etc.) to decide its next action. For example, on an `InvalidNonceError`, the agent knows to retry the transaction. On a `FunctionCallError` with a specific panic message like "ERR_NOT_ENOUGH_FUNDS", it might retry with a larger attached deposit.
-   **State Management**: A more advanced agent would store the `contractId` and other relevant data in a database to track the state of the entities it manages.
-   **Decision Making**: Based on the results of `view` calls, the agent could decide to execute different transactions. For example, if `get_status` returned "needs_update", the agent would trigger the `set_status` call.
