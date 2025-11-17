# HOT Connector

[`@hot-labs/near-connect`](https://github.com/azbang/hot-connector) provides a simple way to integrate with HOT Wallet, which is embedded directly within the Telegram app. `near-kit` supports this through the `fromHotConnect` adapter.

## Step 1: Install Dependencies

```bash
npm install near-kit @hot-labs/near-connect
```

## Step 2: Initialize and Listen for Events

HOT Connector uses an event-driven model. You initialize the connector and then listen for `wallet:signIn` and `wallet:signOut` events to update your application state.

Unlike Wallet Selector, the HOT Connector logic is often self-contained within the component that uses it, as it doesn't rely on a separate modal.

Here is a simplified example component adapted from the example repository that shows how to use the `addGuestbookMessage` universal function.

```typescript
// src/HotConnectorGuestbook.tsx (simplified)
import { NearConnector } from "@hot-labs/near-connect";
import { Near, fromHotConnect } from "near-kit";
import { useEffect, useState } from "react";

// This is the "Universal Code Pattern" function.
// It can be defined in a separate file and imported here.
async function addGuestbookMessage(
  near: Near,
  signerId: string,
  message: string
) {
  return await near
    .transaction(signerId)
    .functionCall(
      "guestbook.near-examples.testnet",
      "add_message",
      { text: message },
      { gas: "30 Tgas" }
    )
    .send();
}

export function HotConnectorGuestbook() {
  const [connector, setConnector] = useState<NearConnector | null>(null);
  const [near, setNear] = useState<Near | null>(null);
  const [accountId, setAccountId] = useState<string | null>(null);
  const [isConnected, setIsConnected] = useState(false);
  const [newMessage, setNewMessage] = useState('');

  useEffect(() => {
    const initConnector = async () => {
      // 1. Initialize the NearConnector
      const nearConnector = new NearConnector({
        network: "testnet", // Or 'mainnet'
        features: { signAndSendTransaction: true, testnet: true },
      });

      // 2. Listen for the 'signIn' event
      nearConnector.on("wallet:signIn", async (data) => {
        // Create Near instance when wallet signs in
        const nearInstance = new Near({
          network: "testnet",
          wallet: fromHotConnect(nearConnector as any), // Cast needed due to type mismatch
        });
        setNear(nearInstance);
        setAccountId(data.accounts[0].accountId);
        setIsConnected(true);
      });

      // 3. Listen for the 'signOut' event
      nearConnector.on("wallet:signOut", async () => {
        // Clear state when wallet signs out
        setNear(null);
        setAccountId(null);
        setIsConnected(false);
      });

      setConnector(nearConnector);
    };

    initConnector();
  }, []);

  const connectWallet = async () => {
    if (connector) {
      await connector.connect(); // Trigger wallet connection
    }
  };

  const disconnectWallet = async () => {
    if (connector) {
      await connector.disconnect(); // Disconnect from wallet
    }
  };

  const handleAddMessage = async () => {
    if (!near || !accountId || !newMessage.trim()) return;

    // Call the universal function
    await addGuestbookMessage(near, accountId, newMessage.trim());
    
    setNewMessage('');
    // ... handle success
  };

  if (!isConnected) {
    return (
      <button onClick={connectWallet}>Connect Wallet (HOT)</button>
    );
  }

  return (
    <div>
      <p>Connected as: {accountId}</p>
      <input
        value={newMessage}
        onChange={(e) => setNewMessage(e.target.value)}
        placeholder="Enter your message..."
      />
      <button onClick={handleAddMessage}>Add Message</button>
      <button onClick={disconnectWallet}>Disconnect</button>
    </div>
  );
}
```
