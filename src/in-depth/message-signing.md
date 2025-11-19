# Message Signing (Authentication)

Transactions change the blockchain state and cost gas. Sometimes, you just want to prove **who you are**.

Message Signing (standardized in [NEP-413](https://github.com/near/NEPs/blob/master/neps/nep-0413.md)) allows a user to sign a piece of data with their private key off-chain. This is free, instant, and is the standard way to implement **"Log in with NEAR"**.

## How it Works

1.  **Client:** Generates a random "nonce" and asks the user to sign a specific message.
2.  **Wallet:** Shows the message to the user. If approved, it returns a cryptographic signature.
3.  **Backend:** Verifies the signature against the user's public key and checks the nonce to prevent replay attacks.

## 1. The Client (Frontend)

Use `near.signMessage` to request a signature.

You **must** generate a random nonce. This ensures that a captured signature cannot be re-used by an attacker later.

```typescript
import { Near, generateNep413Nonce } from "near-kit"

// 1. Generate a 32-byte random buffer
const nonce = generateNep413Nonce()

// 2. Request Signature
const signedMessage = await near.signMessage({
  message: "Log in to MyApp", // What the user sees
  recipient: "myapp.com", // Your app identifier (prevents phishing)
  nonce: nonce,
})

// 3. Send to Backend
await fetch("/api/login", {
  method: "POST",
  body: JSON.stringify({
    signedMessage,
    nonce: Array.from(nonce), // Send the raw nonce bytes
  }),
})
```

```admonish tip title="Handling Binary Data in JSON"
`JSON.stringify` cannot handle raw `Uint8Array` buffers. Note how we convert `nonce` to `Array.from(nonce)` in the example above. If you try to send the raw buffer, the backend will receive an empty object.
```

## 2. The Server (Backend)

On your server, use `verifyNep413Signature`. This function performs the cryptographic math to ensure the signature is valid for the claimed account.

```typescript
import { verifyNep413Signature } from "near-kit"

app.post("/api/login", (req, res) => {
  const { signedMessage, nonce } = req.body

  // Reconstruct nonce buffer from the JSON array
  const nonceBuffer = Buffer.from(nonce)

  // 1. Verify Cryptography & Content
  const isValid = verifyNep413Signature(signedMessage, {
    message: "Log in to MyApp", // Must match exactly what was signed
    recipient: "myapp.com", // Must match YOUR app
    nonce: nonceBuffer, // Must match the nonce sent
  })

  if (!isValid) {
    return res.status(401).send("Invalid Signature")
  }

  // 2. (Recommended) Check for Replays
  // if (db.seenNonces.has(nonceBuffer.toString('hex'))) ...

  // 3. Success!
  console.log(`User verified: ${signedMessage.accountId}`)
  res.send({ token: "session_token_123" })
})
```

```admonish warning title="Security Critical: Replay Attacks"
Cryptographic verification alone is not enough! If you do not check if the `nonce` has been used before, an attacker who intercepts the signed message can "replay" it to your server to log in as the user again.

**Always store used nonces in your database (with an expiration time) and reject duplicates.**
```

## Type Definition

The `SignedMessage` object returned by the client and sent to the server looks like this:

```typescript
type SignedMessage = {
  accountId: string // "alice.near"
  publicKey: string // "ed25519:..."
  signature: string // Base64 encoded signature
}
```
