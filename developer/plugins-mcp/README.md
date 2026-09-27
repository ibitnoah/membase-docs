---
description: The same memory inside ChatGPT, Claude, Cursor, Codex, Grok, Kimi and any MCP client. One server URL, a consent screen, no code.
---

# Plugins Overview

Membase is a standard remote **MCP server**:

```
https://api.app.membase.io/mcp-http
```

Streamable HTTP, plain JSON responses, no trailing slash. It serves the same operations as
the REST API under the same names ([API Reference](../api-reference.md)), so a model that
learned `search_memories` in one client knows it in every other. An AI app that connects to
it reads the person's Memories; nothing is copied into the app.

## Two ways in

| | For | How it authenticates | What it gets |
|---|---|---|---|
| **MCP** | an AI app the person uses: Claude, ChatGPT, Cursor, Codex, Grok, Kimi, VS Code, Windsurf… | the app discovers OAuth, the browser opens Membase's consent screen, the person ticks the Memories the app may use and whether it may read their profile | `list_containers`, `search_memories` and, when ticked, `get_profile`. Read-only by design. |
| **The skill** | an AI that reads pages and runs commands: Claude Code, Codex, an agent framework | a developer key the person minted | the key's access level, up to Full access |

Both are ordinary bindings on the account: the person can narrow or revoke them live on the
**Connect** page, and the change applies on the credential's next call.
[Access Control](access-control.md) has the rules.

### The skill, in one sentence

Tell the AI:

> Install Membase from https://www.app.membase.io/skill using key mbk_…

That address is the skill's `SKILL.md` with an install preamble. The AI fetches the folder,
keeps the key as `MEMBASE_API_KEY` and calls the REST API with it. Connect's **Skills** tab
writes the sentence for you, key included. An AI that already has Membase MCP tools uses
those and keeps the skill for its rules.

## The pages

| Page | For |
|---|---|
| [Membase MCP](membase-mcp.md) | the server itself: discovery, the header alternative, what `tools/list` shows, the ready-made config files, and every client not listed below |
| [Browser Extension](browser-extension.md) | Unibase Memory, which captures conversations with other assistants and hands them to a Memory as a source |
| [ChatGPT](chatgpt.md) | Developer mode, the plugin dialog, per-chat enabling |
| [Claude / Claude Code](claude.md) | the custom connector, `claude mcp add`, the plugin install |
| [Codex](codex.md) | `codex mcp add`, the IDE extension |
| [Cursor](cursor.md) | `mcp.json`, project or global |
| [Grok / Kimi](grok-kimi.md) | Grok's custom connector, Kimi Code CLI |
| [Access Control](access-control.md) | consent vs key, reach, the profile tick, confirmation, revocation |
| [Troubleshooting](troubleshooting.md) | every word and status a client can show, and what to do |

## What connecting looks like

1. The person pastes the URL into the app, or runs the app's `mcp add` command.
2. The app answers with a browser window: Membase's consent card, listing the Memories with a tick each and **Let it know about you** for the profile.
3. The person approves. The app's tile on Connect says *Authorized*; its page lists what it **Uses**.
4. In a chat, the person asks *"List my Membase containers."* An empty list means no Memory is switched on for this connection yet.

Connecting is the account's, once. Which Memories an app uses is the Memory's, and the
switch on the app's page and the switch on the Memory's **Use in** card are the same switch.
