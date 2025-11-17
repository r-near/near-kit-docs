# Wallet Selector

[`@near-wallet-selector`](https://github.com/near/wallet-selector) is a popular library for providing a modal that lets users choose from many different wallets. `near-kit` integrates with it seamlessly via the `fromWalletSelector` adapter.

## Step 1: Install Dependencies

```bash
npm install near-kit @near-wallet-selector/core @near-wallet-selector/modal-ui @near-wallet-selector/meteor-wallet
```

## Step 2: Set up a Wallet Context Provider (Recommended for React)

For React applications, it's highly recommended to centralize your wallet logic in a Context Provider. This allows any component to easily access the `Near` instance and wallet connection status.

Here's an example `WalletProvider.tsx` (adapted from the example repository):

```typescript
// src/WalletProvider.tsx
import React, { createContext, useContext, useEffect, useState } from 'react';
import { setupWalletSelector } from '@near-wallet-selector/core';
import { setupModal } from '@near-wallet-selector/modal-ui';
import { setupMeteorWallet } from '@near-wallet-selector/meteor-wallet'; // Example wallet module
import { Near, fromWalletSelector } from 'near-kit';

interface WalletContextType {
  near: Near | null;
  accountId: string | null;
  isConnected: boolean;
  connectWallet: () => void;
  disconnectWallet: () => void;
}

const WalletContext = createContext<WalletContextType | undefined>(undefined);

export function useWallet() {
  const context = useContext(WalletContext);
  if (context === undefined) {
    throw new Error('useWallet must be used within a WalletProvider');
  }
  return context;
}

export function WalletProvider({ children }: { children: React.ReactNode }) {
  const [near, setNear] = useState<Near | null>(null);
  const [accountId, setAccountId] = useState<string | null>(null);
  const [isConnected, setIsConnected] = useState(false);
  const [selector, setSelector] = useState<any>(null);
  const [modal, setModal] = useState<any>(null);

  useEffect(() => {
    const initWallet = async () => {
      try {
        // 1. Setup wallet selector with desired modules
        const walletSelector = await setupWalletSelector({
          network: 'testnet', // Or 'mainnet'
          modules: [setupMeteorWallet()], // Add other wallets like setupMyNearWallet()
        });

        // 2. Setup the modal UI
        const walletModal = setupModal(walletSelector, {
          contractId: 'guestbook.near-examples.testnet', // Contract your dApp interacts with
        });

        setSelector(walletSelector);
        setModal(walletModal);

        // 3. Listen for wallet state changes
        const unsubscribe = walletSelector.store.observable.subscribe(async (state: any) => {
          if (state.accounts.length > 0 && state.accounts[0]) {
            const wallet = await walletSelector.wallet();
            if (wallet) {
              // Create a Near instance using the fromWalletSelector adapter
              const nearInstance = new Near({
                network: 'testnet',
                wallet: fromWalletSelector(wallet),
              });

              setNear(nearInstance);
              setAccountId(state.accounts[0].accountId);
              setIsConnected(true);
            }
          } else {
            // User disconnected or no account selected
            setNear(null);
            setAccountId(null);
            setIsConnected(false);
          }
        });

        // Check initial state in case user is already signed in
        const state = walletSelector.store.getState();
        if (state.accounts.length > 0 && state.accounts[0]) {
          const wallet = await walletSelector.wallet();
          if (wallet) {
            const nearInstance = new Near({
              network: 'testnet',
              wallet: fromWalletSelector(wallet),
            });
            setNear(nearInstance);
            setAccountId(state.accounts[0].accountId);
            setIsConnected(true);
          }
        }

        return () => unsubscribe(); // Cleanup subscription
      } catch (error) {
        console.error('Failed to initialize wallet:', error);
      }
    };

    initWallet();
  }, []);

  const connectWallet = async () => {
    if (modal) {
      modal.show(); // Show the wallet selection modal
    }
  };

  const disconnectWallet = async () => {
    if (selector) {
      const wallet = await selector.wallet();
      if (wallet) {
        await wallet.signOut(); // Sign out from the connected wallet
      }
    }
  };

  const value: WalletContextType = {
    near,
    accountId,
    isConnected,
    connectWallet,
    disconnectWallet,
  };

  return (
    <WalletContext.Provider value={value}>
      {children}
    </WalletContext.Provider>
  );
}
```

## Step 3: Use the Wallet in Your Components

Once the `WalletProvider` is set up, any child component can use the `useWallet` hook to access the `Near` instance and wallet status. The component can then call your universal business logic function.

```typescript
// src/WalletSelectorGuestbook.tsx (simplified)
import { useWallet } from './WalletProvider';
import type { Near } from 'near-kit';

// This is the "Universal Code Pattern" function from the previous page.
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

export function WalletSelectorGuestbook() {
  const { near, accountId, isConnected, connectWallet, disconnectWallet } = useWallet();
  const [newMessage, setNewMessage] = useState('');

  const handleAddMessage = async () => {
    if (!near || !accountId || !newMessage.trim()) return;

    // Call the universal function
    await addGuestbookMessage(near, accountId, newMessage.trim());

    setNewMessage('');
    // ... reload messages
  };

  if (!isConnected) {
    return (
      <button onClick={connectWallet}>Connect Wallet</button>
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
