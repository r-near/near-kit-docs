# Getting Started

Welcome to `near-kit`, the modern TypeScript SDK for building on the NEAR blockchain! This guide will walk you through setting up your first project, connecting to the NEAR testnet, and sending your first transaction.

## Prerequisites

1.  **[Node.js](https://nodejs.org/) or [Bun](https://bun.sh/):** This library works in both server-side and browser environments. For this guide, we'll use a Node.js/Bun environment.
2.  **A NEAR Testnet Account:** You'll need an account to sign and send transactions. If you don't have one, you can create one at the [official NEAR Wallet](https://wallet.testnet.near.org/).
3.  **A Private Key:** To sign transactions, you need a private key. After creating a testnet account, you can find your key in your browser's local storage or in the `~/.near-credentials/` directory if you've used the [NEAR CLI](https://github.com/near/near-cli-rs).

## Installation

First, add `near-kit` to your project:

```bash
bun install near-kit
```

## Your First Script

Let's write a script that connects to the testnet, reads data from a contract, and then sends a transaction to it.

Create a file named `index.ts`:

```typescript
import { Near } from "near-kit"

async function main() {
  // --- 1. Set up your credentials ---
  // NOTE: Do not hardcode private keys in production! Use environment variables.
  const YOUR_ACCOUNT_ID = "your-account.testnet"
  const YOUR_PRIVATE_KEY = "ed25519:...." // Your testnet private key

  if (YOUR_ACCOUNT_ID === "your-account.testnet") {
    console.warn(
      "Please replace YOUR_ACCOUNT_ID and YOUR_PRIVATE_KEY in this example."
    )
    return
  }

  // --- 2. Initialize the client ---
  // Connect to the testnet and provide your private key for signing.
  const near = new Near({
    network: "testnet",
    privateKey: YOUR_PRIVATE_KEY,
    defaultSignerId: YOUR_ACCOUNT_ID,
  })

  console.log(`Initialized client for account [${YOUR_ACCOUNT_ID}]`)

  // --- 3. View a contract (read-only, no cost) ---
  // Let's check the messages on the guest book contract.
  const messages = await near.view(
    "guestbook.near-examples.testnet",
    "get_messages",
    {}
  )
  console.log("Latest messages on the guest book:", messages)

  // --- 4. Send a transaction (write) ---
  // Now, let's add our own message to the guest book.
  console.log("Sending a transaction to add a message...")

  const result = await near
    .transaction(YOUR_ACCOUNT_ID)
    .functionCall(
      "guestbook.near-examples.testnet",
      "add_message",
      { text: "Hello from near-kit!" },
      { gas: "30 Tgas" } // Attach 30 Tgas to the call
    )
    .send() // Signs and sends the transaction

  console.log("✅ Transaction sent successfully!")
  console.log("Transaction Hash:", result.transaction.hash)
}

main().catch(console.error)
```

```admonish warning title="Security Best Practice"
Do not hardcode private keys in production applications! Use a secure method like environment variables or a secret management service to handle your keys.
```

### Running the Script

Replace the placeholder credentials and run the file:

```bash
bun run index.ts
```

You should see output confirming the client initialization, the number of messages, and finally the success message with your transaction hash. Congratulations, you've just interacted with the NEAR blockchain!

## What's Next?

Now that you've sent your first transaction, let's dive into the [core concepts](./core-concepts/near-instance.md) that make `near-kit` powerful and easy to use.
