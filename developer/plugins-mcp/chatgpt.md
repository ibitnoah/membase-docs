---
description: Membase as a ChatGPT plugin, through Developer mode and the MCP server dialog, with OAuth consent in the browser.
---

# ChatGPT

ChatGPT connects to Membase as a custom MCP server. It needs **Developer mode**, which is
available on the web app for Pro, Plus, Business, Enterprise and Education plans; a
workspace admin can turn it off.

## Set up

1. **Settings → Security and login → Developer mode**: enable it.
2. **Settings → Plugins → +**. Fill the dialog:
   * a name (*Membase*) and a description;
   * **Connection**: *MCP server*;
   * the full URL: `https://api.app.membase.io/mcp-http`;
   * **Authentication**: *OAuth*. Leave Client ID and Secret empty: ChatGPT registers itself with the server.
3. Accept the trust prompt and press **Create**.
4. The browser jumps to Membase's consent screen. Tick the Memories ChatGPT may use, and **Let it know about you** if it may read the profile. Approve.
5. In a chat, enable the plugin under **+ → More → Developer mode**. This is per session.

Then ask *"List my Membase containers."* An empty list means no Memory is switched on for
this connection yet; switch one on under **Uses** on ChatGPT's page in Connect, or under
**Use in** on the Memory.

## What ChatGPT can do

Read. `list_containers`, `search_memories` and, when ticked, `get_profile`. It cannot add,
delete or forget anything, and cannot see a Memory that is not switched on for it. A
destructive request answers with a sentence for ChatGPT to relay, never with an action.

## Two things worth knowing

* ChatGPT registers itself under the same client name as the Codex CLI. Membase tells them apart and shows them as two tiles on Connect; do not be surprised to see both after setting up both.
* If the last step of the flow lands on the Membase app instead of returning to ChatGPT, the authorization still completed; go back to the ChatGPT tab.

## Removing

Delete the plugin under **Settings → Plugins**, and **Disconnect…** on ChatGPT's page in
Membase's Connect. ChatGPT does not tell the server when a plugin is removed.

## If something looks wrong

| It says | What it means | Do |
|---|---|---|
| no **Plugins** entry in Settings | Developer mode is off, or the plan does not include it | Settings → Security and login → Developer mode |
| the plugin is created but never appears in a chat | it is not enabled for this session | **+ → More → Developer mode**, tick it |
| *Uses nothing yet* on Connect | authorized, no Memory switched on | switch one on |
| the consent screen never opens | the URL has a trailing slash or a typo | paste `https://api.app.membase.io/mcp-http` exactly |
