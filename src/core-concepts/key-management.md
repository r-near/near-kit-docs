# Key Management

`near-kit` provides flexible options for managing the cryptographic keys used to sign transactions. You can choose the best key store for your environment, whether it's a server-side application, a command-line script, or a temporary testing environment.

Keys can be provided to the `Near` instance in several ways, with the following priority:

1.  **Per-Transaction Signer**: Using `.signWith()` on a `TransactionBuilder` instance.
2.  **Wallet Adapter**: If a `wallet` is provided, all signing is delegated to the user's browser wallet.
3.  **Custom Signer**: A `signer` function in the `Near` constructor.
4.  **Private Key**: A single `privateKey` in the `Near` constructor.
5.  **Key Store**: A `keyStore` instance that can manage keys for multiple accounts.

## Key Stores

A Key Store is an object responsible for storing and retrieving key pairs for different accounts.

### `InMemoryKeyStore`

This is the default key store if no other is provided. It stores keys in memory, meaning they are lost when the process exits. It's perfect for:

-   Quick scripts
-   Testing environments
-   Frontend applications where keys are managed temporarily

```typescript
import { InMemoryKeyStore, Near } from "near-kit";

// Create an empty in-memory store
const keyStore = new InMemoryKeyStore();

// Add a key programmatically
const keyPair = parseKey("ed25519:...");
await keyStore.add("alice.testnet", keyPair);

const near = new Near({
  network: "testnet",
  keyStore,
});

// You can also initialize it with keys directly
const preloadedKeyStore = new InMemoryKeyStore({
  "alice.testnet": "ed25519:...",
  "bob.testnet": "ed25519:...",
});
```

### `FileKeyStore`

```admonish note title="Node.js Only"
This key store interacts with the local file system and is only available in Node.js or Bun environments.
```

This key store reads and writes key files in a format compatible with `near-cli`. It's the ideal choice for server-side applications and scripts that need persistent access to keys.

Keys are stored in `~/.near-credentials/{networkId}/{accountId}.json`.

```typescript
import { FileKeyStore } from "near-kit/keys"; // Note the sub-path import
import { Near } from "near-kit";

// This will use keys from ~/.near-credentials/testnet/
const keyStore = new FileKeyStore("~/.near-credentials", "testnet");

const near = new Near({
  network: "testnet",
  keyStore,
});

// Now you can sign for any account that has a key file in that directory
await near.transaction("my-account.testnet").transfer("bob.testnet", "1 NEAR").send();
```

### `NativeKeyStore`

```admonish note title="Node.js Only"
This key store interacts with the operating system's native credential manager and is only available in Node.js or Bun environments.
```

For the highest level of security on a server or local machine, the `NativeKeyStore` uses the operating system's native credential manager:

-   **macOS**: Keychain Access
-   **Windows**: Credential Manager
-   **Linux**: Secret Service API (like GNOME Keyring or KDE Wallet)

Keys are encrypted by the OS and are often protected by the user's login password or biometrics.

```typescript
import { NativeKeyStore } from "near-kit/keys"; // Note the sub-path import
import { Near } from "near-kit";

// Keys will be stored securely in the OS keyring under the "NEAR Credentials" service
const keyStore = new NativeKeyStore();

const near = new Near({
  network: "testnet",
  keyStore,
});

// The first time you add a key, the OS may prompt for a password
await keyStore.add("my-secure-account.testnet", keyPair);
```

Due to its reliance on native system libraries, this key store is only available in Node.js environments.

### `RotatingKeyStore`

For applications that need to send many concurrent transactions from a single account, `RotatingKeyStore` eliminates nonce collisions by rotating through multiple access keys in round-robin fashion.

```admonish tip title="High-Throughput Applications"
Use `RotatingKeyStore` when you need to send dozens or hundreds of concurrent transactions from one account. This is ideal for bots, batch processors, and high-volume applications.
```

**How it works:**
- Each `transaction()` call uses the next key in rotation
- Each key has independent nonce tracking via `NonceManager`
- No nonce collisions = 100% success rate for concurrent transactions

```typescript
import { RotatingKeyStore, Near } from "near-kit";

// Initialize with multiple keys for one account
const keyStore = new RotatingKeyStore({
  "alice.testnet": [
    "ed25519:key1...",
    "ed25519:key2...",
    "ed25519:key3...",
  ],
});

const near = new Near({
  network: "testnet",
  keyStore,
});

// Send 100 concurrent transactions - all succeed!
await Promise.all(
  Array(100).fill(0).map((_, i) =>
    near.transaction("alice.testnet")
      .functionCall("contract.testnet", "process", { id: i })
      .send()
  )
);
```

**Setting up multiple keys on-chain:**

Before using `RotatingKeyStore`, you need to add the additional access keys to your account:

```typescript
import { Near, generateKey } from "near-kit";

const near = new Near({
  network: "testnet",
  privateKey: "ed25519:existing-key...",
  defaultSignerId: "alice.testnet",
});

// Generate and add 2 more keys
const key2 = generateKey();
const key3 = generateKey();

await near.transaction("alice.testnet")
  .addKey(key2.publicKey.toString(), { type: "fullAccess" })
  .addKey(key3.publicKey.toString(), { type: "fullAccess" })
  .send();

// Now use all 3 keys with RotatingKeyStore
const keyStore = new RotatingKeyStore({
  "alice.testnet": [
    "ed25519:existing-key...",
    key2.secretKey,
    key3.secretKey,
  ],
});
```

**Advanced usage:**

```typescript
// Add keys dynamically
const keyStore = new RotatingKeyStore();
await keyStore.add("alice.testnet", keyPair1);
await keyStore.add("alice.testnet", keyPair2); // Appends to rotation

// Query rotation state
const allKeys = await keyStore.getAll("alice.testnet");
console.log(`Account has ${allKeys.length} keys in rotation`);

const currentIndex = keyStore.getCurrentIndex("alice.testnet");
console.log(`Next key will be index ${currentIndex % allKeys.length}`);

// Reset rotation to start from first key
keyStore.resetCounter("alice.testnet");
```