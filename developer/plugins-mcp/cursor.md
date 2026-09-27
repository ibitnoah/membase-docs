---
description: Membase in Cursor through mcp.json, in one project or globally, with consent or a developer key.
---

# Cursor

`.cursor/mcp.json` in the project, or `~/.cursor/mcp.json` for every project:

```json
{ "mcpServers": { "membase": { "url": "https://api.app.membase.io/mcp-http" } } }
```

Enable it under **Customize → MCP** and complete the OAuth prompt: the browser opens
Membase's consent screen, where you tick the Memories Cursor may use and, if it may read the
profile, **Let it know about you**.

With a developer key instead of consent, add the header; the key's level applies, up to
Full access:

```json
{ "mcpServers": { "membase": {
    "url": "https://api.app.membase.io/mcp-http",
    "headers": { "Authorization": "Bearer mbk_…" } } } }
```

The key's page in Connect › Developer keys writes this block with the key filled in. Keep a
file that holds a key out of version control; the consent form of the file is safe to
commit.

## The skill

Cursor's agent reads pages and runs commands. Paste the sentence from Connect's **Skills**
tab and it installs the skill folder in the project, points to it from the rules file and
keeps the key. With the MCP tools already present it uses those and keeps the skill for its
rules.

## What Cursor can do

With consent: `list_containers`, `search_memories` and, when ticked, `get_profile`. With a
key: the key's level. In the agent, ask *"List my Membase containers"* to check; an empty
list means no Memory is switched on for this connection.

## If something looks wrong

| It says | What it means | Do |
|---|---|---|
| the server shows red under MCP | the URL is wrong, or consent was not completed | check the URL has no trailing slash; retry the prompt |
| the tools appear but every call is refused | the key in `headers` is expired or revoked | rotate it on the key's page and update the file |
| an empty container list | connected, no Memory switched on | **Uses** on Cursor's page in Connect |
| a project file with a key was committed | the token is public | **Revoke…** it now and mint another |
