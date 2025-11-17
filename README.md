# near-kit Documentation

Documentation site for near-kit - a TypeScript library for interacting with NEAR.

**Live site:** [kit.near.tools](https://kit.near.tools)

## Local Development

This documentation is built with [mdBook](https://rust-lang.github.io/mdBook/).

### Prerequisites

- [Rust and Cargo](https://www.rust-lang.org/tools/install)
- mdBook and mdbook-admonish

### Setup

```bash
# Install mdBook
cargo install mdbook

# Install mdbook-admonish preprocessor
cargo install mdbook-admonish
mdbook-admonish install .
```

### Build and Serve

```bash
# Build the documentation
mdbook build

# Serve locally with live reload
mdbook serve
```

The site will be available at `http://localhost:3000`.

## Deployment

This site automatically deploys to Cloudflare Pages at [kit.near.tools](https://kit.near.tools) when changes are pushed to the `main` branch.
