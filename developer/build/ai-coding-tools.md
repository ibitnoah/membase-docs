---
title: Build with an AI coding assistant
description: "Ask a coding assistant to build a Membase integration, with checks for learning completion, permissions and search failures."
---

# Build with an AI coding assistant

Use this recipe when a coding assistant is implementing Membase in your application.
To give the coding assistant your memory for its own work, follow its client guide under
[Connect your AI](https://noah-gao.gitbook.io/membase-user-guide/connect) instead.

Install the Membase skill using your client's setup guide
([Claude Code](https://noah-gao.gitbook.io/membase-user-guide/connect/clients/claude-code), [Codex](https://noah-gao.gitbook.io/membase-user-guide/connect/clients/codex),
[Cursor](https://noah-gao.gitbook.io/membase-user-guide/connect/clients/cursor), [Kimi Code](https://noah-gao.gitbook.io/membase-user-guide/connect/clients/kimi-code)). Then give it the
integration task below.

## Let the agent build the integration

The same skill carries the API reference, the SDK guide, the MCP setup and the use cases, so an
agent that has it can write the integration for you. Install it once, then prompt:

> Add Membase memory to this app. Read the membase skill first. Use the SDK; read the profile
> once per session when granted, search before answering about past work, and save only what
> the user supplied. Wait for learning before searching newly added documents and check
> `containers[].error`. Never pass `confirm` on your own. Keep the key in the environment.

What the skill teaches, so you can check the result:

| The agent should | Because |
|---|---|
| read `get_profile` once, near the start, when granted | it is small and standing; the memory of who the user is |
| call `search_memories` before answering about past work, decisions or preferences | search is retrieval inside the user's own container |
| cite `container_name` | every hit says which Memory it came from |
| use `add_memory` for a fact, `add_document` for text or a URL, with `custom_id` | documents are raw material until the Memory's run; the same `custom_id` twice is a no-op |
| pass `confirm=true` only after the person agreed | a delete or forget without it answers with a `how` sentence, not an action |
| treat `403` as withdrawn access and stop | the owner narrowed or revoked the key on Connect |
| use a suitable timeout and inspect `containers[].error` | a cold runtime or failed search must not be reported as an empty Memory |

The skill lives at `plugin/skills/membase/` in the product repository and is served as a folder
at `https://www.app.membase.io/plugin/skills/membase/` and as a zip at
`https://www.app.membase.io/plugin/membase-skill.zip`.

## Without the skill

Any agent can be handed the OpenAPI document instead and asked to generate a client:

```bash
curl -s https://www.app.membase.io/plugin/openapi/agent-protocol.json -o agent-protocol.json
npx openapi-typescript agent-protocol.json -o membase.d.ts
```

Three requests cover most integrations: `GET /v1/profile`, `POST /v1/search` with `{"q": …}`
and `POST /v1/memories` with `{"content": …}`, each with the bearer header.
[API reference](api-reference.md) has every operation.
