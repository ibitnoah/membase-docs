# Use your memory from code

A **developer key** is your own credential for a script, a server or an SDK. It reaches your
memory the way a connected app does — the same tools, over the memories you tick — at an access
level you choose, and you can narrow or revoke it on the same card at any time.

Connected apps (Claude, ChatGPT, Cursor…) do not need one: they connect through [Connect](../pages/connect.md)
and approve on the consent screen. A developer key is for code you run yourself.

## Make a key

![Developer keys](../shots/connect-developer-keys.png)

1. Open **Connect**. The **Developer keys** card is under the apps; **New key** (1) opens the dialog.
2. Press **New key**.
3. Name it after what will hold it, for example *nightly-notes script*.
4. Pick the access:
   - **Read** — list your memories, search them, list what they have read.
   - **Read & write** — also add a fact or a document.
   - **Manage** — also delete a document or forget a fact. Each of those calls must still say
     `confirm=true`; a key never removes anything on its own.
5. Tick the **memories it may use**. You can change this later on this card.
6. Tick **Let it know about you** if the key may read your profile — the standing facts your
   assistant keeps about you and what changed recently.
7. Press **Make key**. Copy the token now.

> The token is shown once. A lost key is revoked here and made again.

The dialog also shows the two ways to use it: as an MCP server in Claude Code, and from a
script over HTTP.

## Use it

**In Claude Code**, so that Claude Code reads your memory without an OAuth round-trip:

```bash
claude mcp add --transport http --scope user membase https://api.app.membase.io/mcp-http \
  --header "Authorization: Bearer YOUR_KEY"
```

**From a script**, over HTTP:

```bash
curl -s https://api.app.membase.io/v1/search \
  -H "Authorization: Bearer YOUR_KEY" -H 'content-type: application/json' \
  -d '{"q": "what did we decide about pricing in August"}'
```

The API's words map onto the app's:

| In the app | In the API |
|---|---|
| a Memory | a **container** |
| an entry on a Memory's page | a **memory** |
| a file, note or page a Memory read | a **document** |

The complete reference — every call, field and answer — is
[docs/contract/agent-protocol.md](https://github.com/unibaseio/membase-suites/blob/main/docs/contract/agent-protocol.md)
in the repository, and the `membase` plugin (`plugin/`) teaches it to any AI client.

## Narrow or revoke

On the **Developer keys** card each key shows its access, how many memories it reaches, when it
was made and last used. **Revoke** ends it at once: every script holding the token stops
working on its next call. To change which memories a key reaches, use the switches on the
memory's **Use in** card, the same way as for an app.

## If something looks wrong

| It says | What it means | Do |
|---|---|---|
| `403 … may not use that container` | the key does not reach that memory | switch the memory on for the key, or list containers first |
| `403 … not in this agent's capability profile` | the call is above the key's access | make a key with a higher access level |
| `"status": "confirmation_required"` | a delete or forget without `confirm=true` | say `confirm=true` after you have decided; only a developer key can |
| `422 … no model` | your memory cannot run a turn yet | set a model under [AI Setup](../pages/ai-setup.md) |
| *expired* on the card | the key's token has lapsed | revoke it and make a new one |
