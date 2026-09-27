---
description: The MCP server itself. How a client discovers it, the two ways to authenticate, what a connected client sees, the ready-made config files, and the clients without a page of their own.
---

# Membase MCP

```
https://api.app.membase.io/mcp-http
```

One URL for every client. Streamable HTTP, JSON responses, no trailing slash (a client that
appends one is redirected, but not every client follows a redirect on POST). The server
speaks the current MCP specification and advertises its authorization the standard way, so
a client that can add "a remote MCP server with OAuth" can add Membase without a special
step.

## Two ways to authenticate

**OAuth consent**, for the person's own AI apps. An unauthenticated request answers `401`
with `WWW-Authenticate` pointing at `/.well-known/oauth-protected-resource` (RFC 9728). The
metadata names the authorization server and its registration endpoint; the client registers
itself (RFC 7591) and runs the authorization code flow with PKCE. The browser opens Membase's
consent screen; the person ticks the Memories the app may use and whether it may read their
profile; the client receives a token good for that and nothing else.

**A developer key in the header**, for a client that can send a static header and a person
who prefers a key they minted:

```
Authorization: Bearer mbk_…
```

No consent round-trip; the key's access level and reach apply, up to Full access. The
ready-made config files carry no header, so a key never ends up in a public file.

## What a connected client sees

`tools/list` is filtered per caller.

| Credential | Tools offered |
|---|---|
| consent token | `list_containers`, `search_memories`; `get_profile` when the person ticked it |
| developer key, Read | those, plus `list_documents`, `memory_rules` |
| developer key, Read & write | plus `add_memory`, `add_document` |
| developer key, Full access | plus `delete_document`, `forget_memory` |
| an agent or workflow exposure | `ask_agent` or `workflow_invoke` |

A destructive verb from a consent-minted client never executes; it answers
`status: confirmation_required` and the model relays the `how` sentence. Retired tool names
(`memory_recall`, `memory_remember`, `memory_list`…) still answer for clients that carry them
and are hidden from `tools/list` once their replacement is offered.

A search is a turn inside the person's container. The first one after a quiet spell can
take up to a minute; the answer marks in `containers[]` any Memory that could not answer
yet, and the model should say so rather than answer from nothing.

## Ready-made config files

Everything is served by the app, staged from the `plugin/` folder of the product repository:

| File | For |
|---|---|
| `https://www.app.membase.io/plugin/mcp.json` | the server in the `mcpServers` shape most clients read |
| `https://www.app.membase.io/plugin/clients/vscode.mcp.json` | VS Code's `servers` shape |
| `https://www.app.membase.io/plugin/clients/codex.toml` | Codex's `config.toml` snippet |
| `https://www.app.membase.io/plugin/membase-skill.zip` | the skill folder |
| `https://www.app.membase.io/plugin/openapi/agent-protocol.json` | the OpenAPI document |
| `https://www.app.membase.io/skill` | the skill's `SKILL.md` with its install preamble |

## Other clients

The clients with a page of their own are ChatGPT, Claude and Claude Code, Codex, Cursor, and
Grok and Kimi. Everything else takes the same URL.

### VS Code (Copilot agent mode)

Command palette → *MCP: Add Server* → *HTTP* → the URL → complete the OAuth prompt, or in
`.vscode/mcp.json`:

```json
{ "servers": { "membase": { "type": "http", "url": "https://api.app.membase.io/mcp-http" } } }
```

### Windsurf

`~/.codeium/windsurf/mcp_config.json`, in the `mcpServers` shape:

```json
{ "mcpServers": { "membase": { "serverUrl": "https://api.app.membase.io/mcp-http" } } }
```

Then refresh the MCP list in Windsurf's settings and complete the OAuth prompt.

### Devin

```bash
devin mcp add --scope user membase https://api.app.membase.io/mcp-http
devin mcp login membase
```

### Claude Desktop, and any `mcpServers`-shaped client

Merge `https://www.app.membase.io/plugin/mcp.json` into the client's configuration:

```json
{ "mcpServers": { "membase": { "type": "http", "url": "https://api.app.membase.io/mcp-http" } } }
```

### Any other client, or your own

Use the same URL. The server advertises OAuth discovery on a `401`, accepts a bearer
developer key in `Authorization`, and speaks Streamable HTTP. An agent framework without an
MCP client can take `SKILL.md` as instructions and the REST API directly;
[API Integrations](../api-integrations.md) has the shapes.

## Verifying and revoking

The **Connect** page shows every authorized client with its last activity. A client's page
has the per-Memory switch (*Uses*), the credential list (*Approvals*) and **Disconnect…** for
all of them. Removing a connector inside the client does not notify Membase: revoke on
Connect to be sure access has stopped. Revocation takes effect on the credential's next call.
