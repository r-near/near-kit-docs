# For AI Agents

This documentation is available in a raw text format optimized for Large Language Models (LLMs).

If you are using AI coding assistants like **Cursor**, **Windsurf**, **ChatGPT**, or **Claude**, providing this context file significantly reduces hallucinations and ensures the AI uses the modern `near-kit` API instead of the legacy `near-api-js`.

## Available Context Files

### 1. Full Context (Recommended)

Contains the entire documentation book in a single text file. Use this for most coding tasks.

- **URL:** [https://kit.near.tools/llms-full.txt](https://kit.near.tools/llms-full.txt)
- **Filename:** `llms-full.txt`

### 2. Index Only

A condensed table of contents. Use this if you are hitting context window limits and only need the agent to know the library structure.

- **URL:** [https://kit.near.tools/llms.txt](https://kit.near.tools/llms.txt)
- **Filename:** `llms.txt`

---

## How to Use

### In Cursor / Windsurf

1.  Open the **Chat** pane.
2.  Type `@` and choose **Add Link** (or just paste the URL).
3.  Paste: [https://kit.near.tools/llms-full.txt](https://kit.near.tools/llms-full.txt)
4.  The editor will ingest the documentation as a reference file.

### In ChatGPT / Claude

1.  Paste the URL [https://kit.near.tools/llms-full.txt](https://kit.near.tools/llms-full.txt) into the chat.
2.  Add the prompt below.

## Recommended Prompt

When starting a new session, tell the AI explicitly to ignore the old library.

```text
I am building an application using the `near-kit` library.

IMPORTANT: `near-kit` is NOT `near-api-js`.
- Do not use `near-api-js` patterns (like `utils.format` or `account.functionCall`).
- Use the provided `llms-full.txt` context file as the source of truth for API methods and types.
```
