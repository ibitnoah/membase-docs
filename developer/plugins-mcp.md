---
description: The same memory inside ChatGPT, Claude, Cursor, Codex, VS Code and any MCP client. One server URL, a consent screen, no code.
---

# Plugins & MCP

Membase is a standard remote **MCP server**:

```
https://api.app.membase.io/mcp-http
```

Streamable HTTP, plain JSON responses, no trailing slash. It serves the same operations as the
REST API under the same names ([API Reference](api-reference.md)), so a model that learned
`search_memories` in one client knows it in every other.

## Two ways in

| | For | How it authenticates | What it gets |
|---|---|---|---|
| **MCP** | an AI app the user uses (Claude, ChatGPT, Cursor, Codex, VS Code, Windsurf…) | the app discovers OAuth, the browser opens Membase's consent screen, the user ticks the Memories the app may use and whether it may read their profile | `list_containers`, `search_memories` and, when ticked, `get_profile`. Read-only by design. |
| **The skill** | an AI that reads pages and runs commands (Claude Code, an agent framework) | a developer key the user minted | the key's access level, up to Full access |

Both are ordinary bindings on the account: the user can narrow or revoke them live on the
**Connect** page, and the change applies on the credential's next call.

### The skill, in one sentence

Tell the AI:

> Install Membase from https://www.app.membase.io/skill using key mbk_…

That address is the skill's `SKILL.md` with an install preamble. The AI fetches the folder,
keeps the key as `MEMBASE_API_KEY` and calls the REST API with it. Connect's *with a key* tab
writes the sentence for you, key included.

## Set up a client

Every client below takes the same URL. With OAuth the client asks for consent on first use;
with a developer key the header replaces consent.

### Claude Code

```bash
claude mcp add --transport http --scope user membase https://api.app.membase.io/mcp-http
claude mcp login membase
```

or with a developer key:

```bash
claude mcp add --transport http --scope user membase https://api.app.membase.io/mcp-http \
  --header "Authorization: Bearer ${MEMBASE_API_KEY}"
```

`--scope user` makes the server available in every project. Then ask *"List my Membase
containers."* An empty list means no Memory is switched on for this connection yet.

As a plugin (skill and server in one install):

```
/plugin marketplace add unibaseio/membase-suites
/plugin install membase@unibaseio
```

### Claude (claude.ai and desktop)

Customize → Connectors → **+** → *Add custom connector* → paste the URL → finish the
authorization in the browser. Remove it on the same Connectors page, and revoke on Membase's
Connect page too: the client does not tell the server.

### ChatGPT

Settings → *Security and login* → enable **Developer mode** (web; Pro, Plus, Business,
Enterprise or Education). Settings → **Plugins** → **+**: name, description, Connection =
*MCP server*, paste the full URL, Authentication = *OAuth*, leave Client ID and Secret empty
(ChatGPT registers itself), accept the trust prompt, **Create**. The browser jumps to
Membase's consent screen. In a chat, enable the plugin under **+ → More → Developer mode**,
per session.

### Codex

```bash
codex mcp add membase --url https://api.app.membase.io/mcp-http
codex mcp login membase
codex mcp list
```

The IDE extension takes the same server under the gear menu → *MCP servers*, then *Restart
extension* and *Authenticate*; CLI and extension share the configuration.

### Cursor

`.cursor/mcp.json` in the project, or `~/.cursor/mcp.json` globally:

```json
{ "mcpServers": { "membase": { "url": "https://api.app.membase.io/mcp-http" } } }
```

Enable it under Customize and complete the OAuth prompt. With a developer key, add
`"headers": { "Authorization": "Bearer …" }`.

### VS Code (Copilot agent mode)

Command palette → *MCP: Add Server* → *HTTP* → the URL → complete the OAuth prompt, or in
`.vscode/mcp.json`:

```json
{ "servers": { "membase": { "type": "http", "url": "https://api.app.membase.io/mcp-http" } } }
```

### Windsurf / Devin

```bash
devin mcp add --scope user membase https://api.app.membase.io/mcp-http
devin mcp login membase
```

### Any other client

Use the same URL. The server advertises OAuth discovery (RFC 9728) on a 401, accepts a bearer
developer key in `Authorization`, and speaks Streamable HTTP. An agent framework without MCP
can take `SKILL.md` as instructions and the REST API directly.

## Ready-made config files

Everything is served by the app, staged from the `plugin/` folder of the product repository:

| File | For |
|---|---|
| `https://www.app.membase.io/plugin/mcp.json` | the server in the `mcpServers` shape most clients read |
| `https://www.app.membase.io/plugin/clients/vscode.mcp.json` | VS Code's `servers` shape |
| `https://www.app.membase.io/plugin/clients/codex.toml` | Codex's `config.toml` snippet |
| `https://www.app.membase.io/plugin/membase-skill.zip` | the skill folder |
| `https://www.app.membase.io/plugin/openapi/agent-protocol.json` | the OpenAPI document |

## What a connected client sees

`tools/list` is filtered per caller. A consent-minted connection offers `list_containers`,
`search_memories` and, when the user ticked it, `get_profile`; nothing that writes. A
developer-key connection offers the tools of the key's access level. A destructive verb from
a consent-minted client never executes; it answers `status: confirmation_required` and the
model relays the `how` sentence.

Retired tool names (`memory_recall`, `memory_remember`, `memory_list`…) still answer for
clients that carry them and are hidden from `tools/list` once their replacement is offered.

## Verifying and revoking

The **Connect** page shows every authorized client with its last activity. A client's page has
the per-Memory switch (*Uses*), the credential list (*Approvals*) and **Disconnect…** for all
of them. Removing a connector inside the client does not notify Membase: revoke on Connect to
be sure access has stopped. Revocation takes effect on the credential's next call.
