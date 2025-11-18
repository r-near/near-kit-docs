# Introduction

Welcome to `near-kit`!

`near-kit` is a simple, intuitive TypeScript library for interacting with the NEAR Protocol. It's designed to feel like a modern `fetch` library—easy for beginners, but powerful enough for advanced users.

## Core Philosophy

- **Simple things should be simple**: Common operations should be achievable in a single, clear line of code.
- **Type safety everywhere**: The library is written in TypeScript and provides strong type safety and autocompletion to help you catch errors before you run your code.
- **Progressive complexity**: Start with a simple API for basic needs, and then tap into more advanced features as your application requires them.
- **Human-readable APIs**: Use strings like [`"10 NEAR"`](./core-concepts/units.md) and [`"30 Tgas"`](./core-concepts/units.md) and let the library handle the complex unit conversions, reducing a common source of bugs.

## Key Features

- **[Powerful Transaction Builder](./transactions/builder.md)**: A fluent, chainable API for building complex, multi-action atomic transactions.
- **[Full Wallet Support](./frontend-integration/)**: Drop-in integration for popular web wallets via [NEAR Wallet Selector](./frontend-integration/wallet-selector.md) and [HOT Connector](./frontend-integration/hot-connector.md).
- **[Built-in Sandbox](./testing/sandbox.md)**: A local NEAR blockchain for fast, free, and isolated testing of your application and contracts.
- **Advanced Capabilities**: Out-of-the-box support for [meta-transactions (NEP-366)](./dapp-development/delegate-actions.md), [message signing (NEP-413)](./dapp-development/message-signing.md), and [type-safe contract interfaces](./transactions/type-safe-contracts.md).
- **Resilient by Default**: Automatic nonce managment and smart retries for network errors are built-in, so you can write cleaner, more reliable code.
