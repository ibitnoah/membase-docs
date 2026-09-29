---
title: Build with Membase
description: "Start with one working API call sequence, then learn memory operations, integrate your model, and look up SDK and API details."
icon: code
---

# Build with Membase

Use Membase from code to add material to a person's memory and retrieve it for your app.
Calls use that person's developer key and the Memories they grant it. Python, TypeScript
and REST all reach the same service at `https://api.app.membase.io`.

**Start with the [quickstart](api-quickstart.md).** It creates one document, waits for learning
to finish, and searches the result. The example includes the checks needed to distinguish an
empty Memory from a failed search.

## Choose your next task

| Task | Guide |
|---|---|
| Add facts or documents, search, read the profile, forget or delete | [Memory operations](memory-operations.md) |
| Serve multiple people | [Multi-user isolation](multi-user-isolation.md) |
| Configure timeouts, retries or SDK methods | [SDKs](sdk-quickstart.md) |
| Set access and reach, rotate or revoke a credential | [Authentication](authentication.md) |
| Look up an endpoint or response shape | [API reference](api-reference.md) |
| Fix a failed request or unread document | [API troubleshooting](troubleshooting.md) |

## Integrations

| Your application | Guide |
|---|---|
| Claude API | [Claude API](claude-api.md) |
| OpenAI API or a function-calling model | [OpenAI API](openai-api.md) |
| An agent framework that speaks MCP | [MCP frameworks](mcp-frameworks.md) |
| A coding assistant building the integration for you | [AI coding assistants](ai-coding-tools.md) |

If you want to give an existing AI app your memory without building an integration, use
[Connect your AI](https://noah-gao.gitbook.io/membase-user-guide/connect).

## Understand the model

In the API, a **container** is one *Memory* in the app, a **document** is raw material it
reads, and a **memory** is a learned fact or a profile fact. A container is not an end user.

[Platform overview](platform-overview.md) maps these objects to the app.
[How Membase works](how-membase-works.md) explains learning, retrieval and model requirements.
The [engine benchmark report](https://noah-gao.gitbook.io/membase-user-guide/evaluation/benchmarks) is separate from API setup and performance guidance.
