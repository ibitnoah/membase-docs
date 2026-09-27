---
description: Membase in Grok through a custom connector, and in Kimi Code CLI through kimi mcp add. Both take the same URL and the same consent screen.
---

# Grok / Kimi

Both clients add a remote MCP server by URL and authorize it in the browser, so Membase
needs no special step in either. The menus below are the vendors' own and move with their
releases; the Membase side, the URL, the consent screen and the **Authorized** tile on
Connect, is the same as for every other client.

## Grok

Grok (web, iOS and Android) takes custom MCP servers as **connectors**.

1. Open `grok.com/connectors`.
2. **New Connector → Custom**.
3. Enter the server URL, `https://api.app.membase.io/mcp-http`, and complete the authentication: the browser opens Membase's consent screen. Tick the Memories Grok may use and, if it may read the profile, **Let it know about you**.
4. Grok discovers the tools the server exposes for that consent: `list_containers`, `search_memories` and, when ticked, `get_profile`.

On a Grok Business or Enterprise plan a team admin provisions connectors first. Remove the
connector on the same page, and **Disconnect…** on Grok's page in Membase's Connect; the
client does not tell the server.

Grok's API also takes remote MCP servers as tools on a request; that is a developer
integration rather than a plugin, and it works with a developer key in the server's
`Authorization` header the way [API Integrations](../api-integrations.md) describes for the
Claude API.

## Kimi

**Kimi Code CLI** connects to remote MCP servers over Streamable HTTP:

```bash
kimi mcp add --transport http --auth oauth membase https://api.app.membase.io/mcp-http
kimi mcp auth membase
kimi mcp list
```

`kimi mcp auth` opens the browser on Membase's consent screen and caches the token under
`~/.kimi/mcp-oauth/`. `kimi mcp list` shows the server and its authorization state.

With a developer key instead of consent:

```bash
kimi mcp add --transport http membase https://api.app.membase.io/mcp-http \
  --header "Authorization: Bearer mbk_…"
```

The key's level applies, up to Full access. Kimi Code reads pages and runs commands, so the
skill sentence from Connect's **Skills** tab works as well.

## Checking

In either client, ask *"List my Membase containers."* An empty list means no Memory is
switched on for this connection yet; switch one on under **Uses** on the client's page in
Connect, or under **Use in** on the Memory. The tile on Connect says *Authorized* once the
consent completed, whatever the client's own screen says.
