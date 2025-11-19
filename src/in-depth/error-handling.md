# Error Handling

`near-kit` converts cryptic RPC JSON errors into typed JavaScript Error classes. You can `catch` these errors and handle them logically.

## The Error Hierarchy

All errors extend `NearError`. You can check for specific types using `instanceof`.

```typescript
import {
  FunctionCallError,
  AccountDoesNotExistError,
  NetworkError,
} from "near-kit"

try {
  await near.call("contract.near", "method", {})
} catch (e) {
  if (e instanceof FunctionCallError) {
    // The contract logic failed
    console.log("Panic:", e.panic)
    console.log("Logs:", e.logs)
  } else if (e instanceof AccountDoesNotExistError) {
    // The account isn't real
    console.log(`Account ${e.accountId} not found`)
  } else if (e instanceof NetworkError) {
    // RPC is down
    if (e.retryable) {
      // You might want to try again
    }
  }
}
```

## Panic Messages

When a contract fails, the most important info is the **Panic Message**. `near-kit` extracts this from the deep RPC response and puts it right on `error.panic`.

Common panics include:

- `ERR_NOT_ENOUGH_FUNDS`
- `ERR_INVALID_ARGUMENT`
- `Smart contract panicked: ...`

Use this string to debug or show user-friendly error messages in your UI.
