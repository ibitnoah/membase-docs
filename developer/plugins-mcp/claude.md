---
description: Membase in Claude on the web and desktop as a custom connector, and in Claude Code over MCP, with a developer key, or as a plugin.
---

# Claude / Claude Code

## Claude (claude.ai and the desktop app)

Customize → **Connectors** → **+** → *Add custom connector* → paste
`https://api.app.membase.io/mcp-http` → finish the authorization in the browser. On
Membase's consent screen tick the Memories Claude may use and, if it may read the profile,
**Let it know about you**.

Claude then holds `list_containers`, `search_memories` and, when ticked, `get_profile`, and
nothing that writes. Remove it on the same Connectors page, and **Disconnect…** on Claude's
page in Membase's Connect too: the client does not tell the server.

## Claude Code

Three ways, depending on whether you want consent, a key, or the skill as well.

### Over MCP, with consent

```bash
claude mcp add --transport http --scope user membase https://api.app.membase.io/mcp-http
claude mcp login membase
```

`claude mcp login` is Claude Code 2.1.186 or later; older versions authorize with `/mcp`
inside a session. `--scope user` makes the server available in every project. Then ask
*"List my Membase containers."*

### Over MCP, with a developer key

No OAuth round-trip, and the key's level applies, up to Full access:

```bash
claude mcp add --transport http --scope user membase https://api.app.membase.io/mcp-http \
  --header "Authorization: Bearer ${MEMBASE_API_KEY}"
```

The key's page in Connect › Developer keys writes this command with the key filled in.

### As a plugin: skill and server in one install

```
/plugin marketplace add unibaseio/membase-suites
/plugin install membase@unibaseio
```

The plugin declares the server without a header, so Claude Code authorizes with OAuth on
first use, and installs the skill beside it so Claude Code also knows the rules of the road
(profile first, search before answering, save only what the user supplied, never confirm on
its own).

### The skill alone

Paste the sentence from Connect's **Skills** tab:

> Install Membase from https://www.app.membase.io/skill using key mbk_…

Claude Code fetches the skill folder into `~/.claude/skills/membase/`, stores the key as
`MEMBASE_API_KEY` in the `env` block of `~/.claude/settings.json`, checks it with one call
and reports which Memories it can use. If it already has the MCP tools it uses those and
keeps the skill for its rules.

## If something looks wrong

| It says | What it means | Do |
|---|---|---|
| `claude mcp list` shows membase as *needs authentication* | consent not completed | `claude mcp login membase`, or `/mcp` in a session |
| *List my containers* answers an empty list | connected, no Memory switched on | **Uses** on Claude Code's page in Connect, or **Use in** on the Memory |
| the skill says it has no access | no key, or the key reaches nothing | Skills tab › **Create key**; or switch a Memory on for the key |
| a write is refused with `403` | the connection is consent-minted (read-only), or the key is Read | use a Read & write key |
| the first search takes a minute | the memory was asleep | nothing; the next one is quick |
