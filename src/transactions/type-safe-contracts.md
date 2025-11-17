# Type-Safe Contracts

While `near.view()` and `near.call()` are convenient, they lack type safety for your contract's specific methods and arguments. `near-kit` solves this with a powerful `near.contract()` method that creates a type-safe JavaScript proxy for your smart contract.

This gives you:
- **Autocompletion** for method names.
- **Type checking** for arguments and return values.
- A clean, object-oriented way to interact with your contract.

## 1. Define Your Contract's Interface

First, create a TypeScript type that describes your contract's view and call methods. You'll need two nested objects: `view` for read-only methods and `call` for change methods.

```typescript
import type { Contract } from "near-kit";

// Define the method signatures for the guestbook contract
type GuestbookContract = Contract<{
  view: {
    get_messages: (args: {
      from_index?: number;
      limit?: number;
    }) => Promise<Array<{ sender: string; text: string }>>;
    total_messages: () => Promise<number>;
  };
  call: {
    add_message: (args: { text: string }) => Promise<void>;
  };
}>;
```

## 2. Create the Contract Proxy

Next, use `near.contract<T>()` to create a typed proxy object. Pass your contract type as the generic parameter and the `contractId` as the argument.

```typescript
const GUESTBOOK_CONTRACT_ID = "guestbook.near-examples.testnet";

// Create a typed contract instance
const guestbook = near.contract<GuestbookContract>(GUESTBOOK_CONTRACT_ID);
```

## 3. Call Methods with Full Type Safety

Now you can call methods on the `guestbook` object with full autocompletion and type safety. `near-kit` automatically maps these calls to the appropriate `near.view()` or `near.call()` under the hood.

### Calling a View Method

View methods are available under the `.view` property.

```typescript
// Autocompletes `get_messages` and `total_messages`
const messages = await guestbook.view.get_messages({ limit: 10 });

console.log(messages[0].sender); // `sender` is a known string property
```

### Calling a Call Method

Call methods are available under the `.call` property. They automatically receive an optional second `options` parameter for `gas` and `attachedDeposit`.

```typescript
// Autocompletes `add_message`
await guestbook.call.add_message(
  { text: "Hello from a type-safe contract!" }, // Argument is type-checked
  { gas: "30 Tgas", attachedDeposit: "1 yocto" } // Options are also available
);
```

If you try to pass incorrect arguments or access a method that doesn't exist, TypeScript will catch the error at compile time, preventing a whole class of runtime bugs.

```typescript
// TypeScript Error: Property 'addMessage' does not exist on type...
await guestbook.call.addMessage({ text: "..." });

// TypeScript Error: Argument of type '{ message: string }' is not
// assignable to parameter of type '{ text: string }'.
await guestbook.call.add_message({ message: "..." });
```