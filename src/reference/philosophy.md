# A Note on Philosophy: Why I Built `near-kit`

Hey there,

If you're reading this, you're probably trying to figure out if this is the right tool for you. So, instead of a formal "Design Philosophy," I thought I'd just tell you why I built `near-kit` and the problems I was trying to solve for myself.

I started this project after feeling a bit of friction building on NEAR. The existing tools were powerful, but I often found myself wrestling with boilerplate, worrying about subtle but costly mistakes, and spending more time on the plumbing than on the actual product. I wanted a toolkit that felt like a sharp, well-balanced chef's knife—something that felt good in my hand and let me focus on the craft.

This led me to a few core ideas that guide every part of `near-kit`.

---

## 1. The Common Path Should Be Easy and Obvious

Building a transaction shouldn't feel like assembling an engine from a diagram. For 90% of use cases, you're just doing a few simple things in a row. The library should reflect that.

My goal was to create a "fluent" API that reads like a story. This led to the `TransactionBuilder`:

```typescript
// I wanted to be able to write code that looked like this:
const result = await near
  .transaction("alice.near")
  .createAccount("bob.near")
  .transfer("bob.near", "10 NEAR")
  .addKey(newPublicKey, { type: "fullAccess" })
  .send()
```

Each step is just another method call. You chain them together, and when you're done, you `.send()`. Behind the scenes, it seamlessly handles tricky parts like nonce management and retries, but you don't have to think about it. It’s predictable and hard to mess up.

---

## 2. Prevent the "Stupid Mistake" Tax

In blockchain, a simple mistake—like forgetting a few zeros—can cost real money. I call this the "stupid mistake tax," and I think a good library should do everything it can to help you avoid paying it.

This is why `near-kit` is so opinionated about units. I can't tell you how many times I've second-guessed myself: "is this `NEAR` or `yoctoNEAR`?"

`near-kit` solves this by forcing you to be explicit. It will throw an error if you pass in a plain number for an amount.

```typescript
// GOOD: This is clear and unambiguous.
near.send("bob.near", "1.5 NEAR")
near.call("contract.id", "method", {}, { gas: "30 Tgas" })

// BAD: This will throw an error, saving you from a potential disaster.
near.send("bob.near", 1.5)
// Error: Ambiguous amount. Did you mean "1.5 NEAR"?
```

It’s a little extra typing, but the peace of mind is worth it. I even took this to the compile-time level using TypeScript, so if you try to pass an invalid key format, your editor will yell at you before you even run the code.

---

## 3. Solve the Whole Problem, Not Just Part of It

My frustration didn't stop at just sending transactions. I also got tired of the friction around testing and wallet integration. A library should help you through the entire development lifecycle.

**For Testing:** I spent too much time managing third-party libraries with incompatible versions, just to run integration tests. So, I built the `Sandbox`. It’s a simple class that programmatically starts and stops a local NEAR node for you, right from your test files. This has been a game-changer for my own workflow.

**For dApps:** Connecting to a browser wallet is a very common and often tricky problem. To solve this, I included a `wallets` module with simple adapters for popular libraries like [NEAR Wallet Selector](../frontend-integration/wallet-selector.md) or [HOT Connector](../frontend-integration/hot-connector.md). This turns what can be a multi-hour headache into a single function call.

---

## 4. Clarity and No Magic

A library shouldn't be a "black box." When something goes wrong, you should be able to understand why. I've tried to design `near-kit` to be as transparent as possible, favoring clarity over cleverness.

A great example is how it handles errors. The NEAR RPC can return some cryptic error messages. Instead of just passing those along, `near-kit` systematically parses them into clean, understandable error classes.

So, instead of debugging a generic error like `{"code": -32000, "data": "..."}`, you get a specific, actionable error right in your code:

```typescript
try {
  await near.send("non-existent-account.near", "1 NEAR")
} catch (e) {
  if (e instanceof AccountDoesNotExistError) {
    // Now you can handle this specific case!
    console.log(`The account ${e.accountId} does not exist.`)
  }
}
```

This commitment to clarity makes debugging easier and helps you build more resilient applications, because you have confidence in what your tools are doing.

---

# What `near-kit` Isn't

To help you decide if this is the right fit, it's just as important to know what this library _doesn't_ try to be.

- **It's not an infrastructure-grade backend toolkit.** `near-kit` is designed for dApp developers and script writers. While it's robust for these use cases, if your primary job is building a high-availability backend service (like a complex relayer or an RPC load balancer) that requires fine-grained control over network failover policies, you might need a more low-level, specialized library.

- **It's not a web framework.** `near-kit` is a toolkit for interacting with the NEAR Protocol. It doesn't impose any structure on your application and can be integrated into any project, whether it's built with React, Vue, Svelte, or just plain JavaScript.

- **It's not just a minimal wrapper.** The goal isn't just to make RPC calls. The value of `near-kit` comes from the thoughtful abstractions, safety features, and integrated tools that are designed to make your entire development process smoother and safer.

My hope is that when you use `near-kit`, you feel empowered, confident, and maybe even have a little fun.

Happy building.
