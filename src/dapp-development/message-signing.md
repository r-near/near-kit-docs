# Message Signing (NEP-413)

[NEP-413](https://github.com/near/NEPs/blob/master/neps/nep-0413.md) provides a standard for signing and verifying messages off-chain. This is incredibly useful for proving ownership of an account without requiring a user to submit a transaction and pay for gas. The most common use case is "Sign in with NEAR".

`near-kit` provides helpers for both signing on the client-side and verifying on the server-side.

## The Flow

1.  **Client-Side**: The frontend asks the user to sign a message (e.g., "Login to My Awesome App").
2.  **User Signs**: The user signs the message using their wallet. This is a cryptographic signature and does not cost any gas.
3.  **Client Sends to Server**: The frontend sends the signed message object to a backend API endpoint (e.g., `/login`).
4.  **Server-Side**: The backend verifies that the signature is valid for the given message and the user's account. If valid, the server can issue a session token (e.g., a JWT).

## Client-Side: Signing the Message

You can sign a message using the `near.signMessage()` method. You must provide a `message`, a `recipient` (who the message is for, typically your app's name or domain), and a `nonce` for replay protection.

```typescript
import { Near, generateNep413Nonce } from "near-kit";

// In your frontend application
async function handleLogin(near: Near) {
  // 1. Generate a 32-byte nonce for replay protection
  const nonce = generateNep413Nonce();

  // 2. Ask the user to sign the message
  const signedMessage = await near.signMessage({
    message: "Sign in to My Awesome App",
    recipient: "My Awesome App",
    nonce,
  });

  // 3. Send the signed message and the nonce to your backend
  const response = await fetch("/api/login", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      signedMessage,
      nonce: Array.from(nonce), // Send nonce as an array of numbers
    }),
  });

  if (response.ok) {
    console.log("Successfully logged in!");
    // Handle successful login, e.g., store session token
  } else {
    console.error("Login failed");
  }
}
```

### What is a Nonce?

A nonce ("number used once") is a random 32-byte value used to ensure that a signed message cannot be captured and re-used by an attacker. Your server should store the nonces it has already seen and reject any login attempts that use a nonce more than once.

## Server-Side: Verifying the Signature

On your backend, you need to verify the signature that the client sends. `near-kit` provides the `verifyNep413Signature` utility for this.

```typescript
import { verifyNep413Signature, NearError } from "near-kit";
import type { SignedMessage } from "near-kit";

// In your backend API (e.g., Express.js)
// POST /api/login
// body: { signedMessage: SignedMessage, nonce: number[] }
async function loginEndpoint(req, res) {
  const { signedMessage, nonce } = req.body;

  // 1. Reconstruct the nonce from the array
  const nonceBuffer = Uint8Array.from(nonce);

  // 2. Verify the signature
  const isValid = verifyNep413Signature(signedMessage, {
    message: "Sign in to My Awesome App",
    recipient: "My Awesome App",
    nonce: nonceBuffer,
  });

  if (!isValid) {
    return res.status(401).send("Invalid signature");
  }

  // [Optional but Recommended] Check that the nonce has not been used before
  // ... your nonce storage logic here ...

  // 3. Check that the signer is the expected user
  console.log(`Signature from ${signedMessage.accountId} is valid.`);

  // 4. Issue a session token (e.g., JWT)
  // const token = createSessionToken(signedMessage.accountId);
  // res.json({ token });
  res.send("Login successful!");
}
```

By combining `signMessage` on the frontend and `verifyNep413Signature` on the backend, you can build a secure, gasless authentication system for your dApp.
