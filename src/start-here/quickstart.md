# Quickstart

Let's get you connected to the NEAR Testnet and sending your first transaction in under 2 minutes.

## Prerequisites

- **Node.js** or **Bun** installed.
- A [NEAR Testnet Account](https://testnet.mynearwallet.com/).
- Your account's **Private Key** (starts with `ed25519:...`).

## Installation

```bash
# npm
npm install near-kit

# bun
bun add near-kit

# yarn
yarn add near-kit
```

## Your First Script

Create a file named `index.ts`.

```typescript
import { Near } from "near-kit"

async function main() {
  // 1. Initialize the client
  // In a real app, ALWAYS use environment variables for keys!
  const near = new Near({
    network: "testnet",
    privateKey: "ed25519:...", // Replace with your actual private key
    defaultSignerId: "your-account.testnet", // Replace with your account ID
  })

  console.log("🔗 Connected to Testnet")

  // 2. Read some data (Free!)
  // Let's check the balance of the account we just connected
  const balance = await near.getBalance("your-account.testnet")
  console.log(`💰 Balance: ${balance}`)

  // 3. Write some data (Send a transaction)
  // We will send 1 NEAR to ourselves.
  // Note: "1 NEAR" is a string. No math required.
  console.log("💸 Sending 1 NEAR...")

  const result = await near
    .transaction("your-account.testnet")
    .transfer("your-account.testnet", "1 NEAR")
    .send()

  console.log(`✅ Transaction successful!`)
  console.log(`   Hash: ${result.transaction.hash}`)
}

main().catch(console.error)
```

Run it:

```bash
# using bun
bun run index.ts

# using tsx
npx tsx index.ts
```
