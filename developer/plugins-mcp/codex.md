---
description: Membase in the Codex CLI and IDE extension over MCP, with consent or a developer key.
---

# Codex

## CLI

```bash
codex mcp add membase --url https://api.app.membase.io/mcp-http
codex mcp login membase
codex mcp list
```

`codex mcp login` opens the browser on Membase's consent screen; tick the Memories Codex may
use and, if it may read the profile, **Let it know about you**. `codex mcp list` shows the
server and whether it is authorized.

With a developer key instead of consent, put the snippet Membase serves at
`https://www.app.membase.io/plugin/clients/codex.toml` into `~/.codex/config.toml` and add
the header:

```toml
[mcp_servers.membase]
url = "https://api.app.membase.io/mcp-http"
http_headers = { Authorization = "Bearer mbk_…" }
```

The key's page in Connect › Developer keys carries the same snippet with the key filled in.

## IDE extension

The gear menu → *MCP servers* → add the same server, then *Restart extension* and
*Authenticate*. The CLI and the extension share the configuration, so a server added on
one side appears on the other.

## The skill

Codex reads pages and runs commands, so the skill works too:

> Install Membase from https://www.app.membase.io/skill using key mbk_…

It installs the folder beside `AGENTS.md`, points to it from there, and keeps the key as
`MEMBASE_API_KEY` under `env` in its `config.toml`. With the MCP tools already present it
uses those and keeps the skill for its rules.

## Two clients, one name

Codex and the ChatGPT connector register with Membase under the same client name. Membase
tells them apart and shows two tiles on Connect; each has its own **Uses** switches and its
own **Disconnect…**.

## If something looks wrong

| It says | What it means | Do |
|---|---|---|
| `codex mcp list` shows the server without authorization | consent not completed | `codex mcp login membase` |
| an empty container list | connected, no Memory switched on | **Uses** on Codex's page in Connect |
| a write is refused | the connection is consent-minted (read-only) | switch to a Read & write key in `config.toml` |
| the extension does not see the server | it was added before the extension restarted | *Restart extension*, then *Authenticate* |
