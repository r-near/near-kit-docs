# Using with AI Agents

This documentation is designed to be highly compatible with modern AI coding assistants and Large Language Models (LLMs). To facilitate this, we provide two special, pre-compiled text files that you can use to give an AI agent the full context of the `near-kit` library.

These files are automatically generated and contain parts of this documentation book in a format that is easy for an agent to ingest.

## Available Context Files

When you visit the deployed version of this documentation book, you will find two text files at the root:

-   `/llms.txt`: This file serves as a concise table of contents for the "AI Agent Cookbook" sections of the documentation. It lists the main topics and provides direct links to each section, allowing an agent to quickly navigate the AI-specific content.

-   `/llms-full.txt`: This file contains the *entire content* of the "AI Agent Cookbook" sections, concatenated into a single plain text file. It provides the complete, detailed context of the AI-focused documentation, including patterns, examples, and explanations.

## How to Use

When you start a session with your AI coding assistant (like Gemini, ChatGPT, Claude, etc.), you can provide one of these files as context.

### Recommended Prompt Structure

Here is an example of a good starting prompt to give your agent.

> You are an expert programmer specializing in the NEAR blockchain. You will be helping me build an application using the `near-kit` TypeScript library.
>
> It is very important that you understand that **`near-kit` is NOT `near-api-js`**. It is a new library with a different, more modern API. Do not use any patterns from `near-api-js`.
>
> I am providing you with the official `near-kit` documentation. Please use the information from this file exclusively to answer my questions and write code.
>
> [Attach the `llms.txt` or `llms-full.txt` file here, or provide the URL to it]

### Which File to Choose?

-   **Start with `llms.txt`**. This file provides a quick overview and navigation for the AI Agent Cookbook. It's useful for an agent to understand the structure of the AI-specific documentation.
-   **Use `llms-full.txt`** when the agent needs the full, detailed content of the AI Agent Cookbook. This is the primary file for providing the agent with the actual documentation text.

By providing this context, you can significantly improve the accuracy and quality of the code and explanations you get from an AI agent, ensuring it uses the correct patterns for `near-kit`.