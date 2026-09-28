---
title: AI coding tools
description: "Build with Membase from Claude Code, Cursor, Codex or any agent that reads pages and runs commands: one skill teaches it the API, the SDK and the rules, and the same sentence gives it the user's memory."
---

# AI coding tools

An AI that reads pages and runs commands, Claude Code, Codex, Cursor's agent, Kimi Code, or
your own agent with a shell, needs no integration code from you. One skill does two jobs: it
gives the agent the user's memory, and it teaches the agent how to build on the API.

## Give the agent memory

Tell it:

> Install Membase from https://www.app.membase.io/skill using key mbk_…

That address is the skill's `SKILL.md` with an install preamble. The agent fetches the folder,
keeps the key as `MEMBASE_API_KEY` and calls the REST API with it, at the key's level. Connect's
**Skills** tab writes the sentence with a key filled in. The per-client pages
([Claude Code](https://noah-gao.gitbook.io/membase-user-guide/connect/clients/claude-code), [Codex](https://noah-gao.gitbook.io/membase-user-guide/connect/clients/codex),
[Cursor](https://noah-gao.gitbook.io/membase-user-guide/connect/clients/cursor), [Kimi Code](https://noah-gao.gitbook.io/membase-user-guide/connect/clients/kimi-code)) say where each one keeps
the folder and the key.

## Let the agent build the integration

The same skill carries the API reference, the SDK guide, the MCP setup and the use cases, so an
agent that has it can write the integration for you. Install it once, then prompt:

> Add Membase memory to this app. Read the membase skill first. Use the SDK; read the profile
> once per session, search before answering about the user's past work, save only what the user
> supplied, and never pass `confirm` on your own. Put the key in the environment, not in source.

What the skill teaches, so you can check the result:

| The agent should | Because |
|---|---|
| read `get_profile` once, near the start | it is small and standing; the memory of who the user is |
| call `search_memories` before answering about past work, decisions or preferences | search is retrieval inside the user's own container |
| cite `container_name` | every hit says which Memory it came from |
| use `add_memory` for a fact, `add_document` for text or a URL, with `custom_id` | documents are raw material until the Memory's run; the same `custom_id` twice is a no-op |
| pass `confirm=true` only after the person agreed | a delete or forget without it answers with a `how` sentence, not an action |
| treat `403` as withdrawn access and stop | the owner narrowed or revoked the key on Connect |
| keep the SDK's 90-second timeout on the first search | an idle container wakes on the first request |

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
