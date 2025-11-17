# AI Agent Cookbook: Error Handling

This document provides patterns for handling the typed errors thrown by `near-kit`. A robust agent must be able to catch and interpret these errors to act intelligently.

## Core Concept

All `near-kit` errors extend the base JavaScript `Error`. Use a standard `try...catch` block and check the error's type using `instanceof` to handle specific cases.

---

## Patterns

### Pattern 1: Handling a Failed Contract Call

A `FunctionCallError` is the most common error. It means the transaction was valid, but the contract code panicked during execution.

```typescript
import { Near, FunctionCallError } from "near-kit";
const near = new Near({ network: "testnet", privateKey: "..." });

try {
  await near.call("some-contract.testnet", "method_that_panics", {});
} catch (error) {
  if (error instanceof FunctionCallError) {
    // The contract panicked. This is the most useful error for debugging.
    console.error(`Contract Error in ${error.contractId}`);
    console.error(`- Method: ${error.methodName}`);
    
    // The specific panic message from the contract.
    // This is the most important piece of information.
    console.error(`- Panic: ${error.panic}`); 
    
    // Logs emitted by the contract before it panicked.
    console.error(`- Logs:`, error.logs);
    
    // The full transaction result that led to the error.
    // console.error(`- Result:`, error.result);
  } else if (error instanceof Error) {
    console.error("An unexpected error occurred:", error.message);
  }
}
```

**Actionable Intelligence:** The `error.panic` string (e.g., `"ERR_NOT_ENOUGH_FUNDS"`, `"ERR_INVALID_ARGUMENT"`) is the key. An agent can use this string to understand *why* the call failed and change its strategy (e.g., attach more deposit, correct the arguments).

### Pattern 2: Handling an Account That Doesn't Exist

An `AccountDoesNotExistError` occurs when you try to interact with an account that has not been created.

```typescript
import { Near, AccountDoesNotExistError } from "near-kit";
const near = new Near({ network: "testnet", privateKey: "..." });

try {
  await near.send("non-existent-account.testnet", "1 NEAR");
} catch (error) {
  if (error instanceof AccountDoesNotExistError) {
    console.error(`The account "${error.accountId}" does not exist.`);
    // Agent's next action could be to create the account.
  } else if (error instanceof Error) {
    console.error("An unexpected error occurred:", error.message);
  }
}
```

**Actionable Intelligence:** When this error is caught, the agent knows it must first create the account using a `.createAccount()` action before it can proceed.

### Pattern 3: Handling Network and RPC Errors

A `NetworkError` indicates a problem with the RPC node or the connection to it.

```typescript
import { Near, NetworkError } from "near-kit";
const near = new Near({ network: "testnet", privateKey: "..." });

try {
  await near.getStatus();
} catch (error) {
  if (error instanceof NetworkError) {
    console.error(`A network error occurred: ${error.message}`);
    
    // near-kit automatically retries retryable errors.
    // This flag tells you if the error is considered temporary.
    if (error.retryable) {
      console.log("This was a temporary issue. You can try again later.");
    } else {
      console.log("This seems to be a permanent network issue.");
    }
  } else if (error instanceof Error) {
    console.error("An unexpected error occurred:", error.message);
  }
}
```

**Actionable Intelligence:** The `error.retryable` flag is important. If `true`, the agent can wait and try the exact same operation again. If `false`, it may need to switch to a different RPC endpoint.

### Pattern 4: Handling Invalid Transactions

An `InvalidTransactionError` means the transaction was rejected by the network before execution. This can happen for several reasons, such as using a key with insufficient permissions or a nonce being incorrect (though `near-kit` handles nonce errors automatically).

```typescript
import { Near, InvalidTransactionError } from "near-kit";
const near = new Near({ network: "testnet", privateKey: "..." });

try {
  // Example: Trying to add a full access key with a limited access key
  await near.transaction("signer.testnet").addKey("ed25519:...", { type: "fullAccess" }).send();
} catch (error) {
  if (error instanceof InvalidTransactionError) {
    // The `cause` property often contains the specific reason.
    console.error(`Invalid Transaction: ${error.cause}`);
  } else if (error instanceof Error) {
    console.error("An unexpected error occurred:", error.message);
  }
}
```

**Actionable Intelligence:** The `error.cause` string provides the reason for the failure (e.g., `"InvalidAccessKey"`). The agent must analyze this to understand the constraint it violated.
