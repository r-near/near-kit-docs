# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a documentation site for `near-kit`, a TypeScript library for interacting with the NEAR blockchain. The documentation is built using [mdBook](https://rust-lang.github.io/mdBook/), a Rust-based documentation generator that converts Markdown files into a static HTML site.

**Live site:** https://kit.near.tools

## Build System

### Prerequisites

- Rust and Cargo (for mdBook tools)
- Bun (for deployment)

### Key mdBook Tools

This project uses several mdBook preprocessors and tools:
- `mdbook` (v0.4.52) - Core documentation builder
- `mdbook-admonish` - Adds styled admonition blocks (note, warning, tip, etc.)
- `mdbook-linkcheck` - Validates internal and external links
- `mdbook-llms-txt-tools` - Generates AI-friendly text versions of documentation

### Common Commands

```bash
# Build the documentation (generates output to ./book/html/)
mdbook build

# Serve locally with live reload at http://localhost:3000
mdbook serve

# Run with debug logging
RUST_LOG=debug mdbook build

# Install required tools (if not already installed)
cargo install mdbook
cargo install mdbook-admonish
mdbook-admonish install .
```

## Documentation Structure

Documentation source files are in `src/` and organized by category:

- `start-here/` - Introduction, quickstart, mental model, migration guide
- `essentials/` - Core concepts: reading data, writing data, type-safe contracts
- `dapp-workflow/` - Frontend integration, universal pattern, testing
- `in-depth/` - Advanced topics: error handling, key management, transactions, message signing
- `reference/` - Action reference, configuration, data structures, AI integration

The site structure is defined in `src/SUMMARY.md`, which controls the sidebar navigation.

## Styling and Theme

The site uses Catppuccin themes (Latte for light, Macchiato for dark) with custom CSS:
- `theme/catppuccin.css` - Base theme colors
- `theme/catppuccin-admonish.css` - Admonition styling
- `theme/mdbook-admonish-custom.css` - Custom admonition overrides
- `theme/aha_styles.css` - Additional custom styles
- `theme/custom.css` - Site-specific customizations

Custom admonition directive `reset` is configured in `book.toml`.

## Deployment

The site deploys automatically to Cloudflare Pages (via Workers) at https://kit.near.tools when changes are pushed to the `main` branch.

The deployment workflow (`.github/workflows/deploy.yml`):
1. Installs mdBook tools (with caching for faster builds)
2. Runs `mdbook build`
3. Copies `llms.txt`, `llms-full.txt`, and source markdown files to `book/html/`
4. Deploys to Cloudflare Workers using `bunx wrangler deploy`

Cloudflare Workers configuration is in `wrangler.jsonc` and points to the `./book/html` directory as static assets.

## LLMs.txt Generation

The site generates two AI-friendly text versions:
- `llms.txt` - Condensed version for AI agents
- `llms-full.txt` - Full content version

Both are restricted to the "For AI Agents" section (see `reference/ai-integration.md`) and available at the site root.

## Content Guidelines

When editing documentation:
- All content is Markdown in the `src/` directory
- Use mdbook-admonish syntax for callouts: `> [!note]`, `> [!warning]`, `> [!tip]`, etc.
- The custom `> [!reset]` directive is available for special callouts
- Edit URLs are enabled - each page has an "Edit on GitHub" link
- Internal links use relative paths (e.g., `/start-here/quickstart.md`)
- All admonitions are collapsible by default (configured in `book.toml`)
