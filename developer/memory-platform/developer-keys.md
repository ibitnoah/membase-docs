---
description: The screen where keys are minted, narrowed, rotated and revoked, and what each control does to the next call.
---

# Developer Keys

A **developer key** is the owner's own credential for a script, a server, an SDK or an AI
running the skill. It reaches the memory the way a connected app does, over the Memories the
owner chooses, at an access level they pick, for as long as they say. The model behind it is
on [Authentication & Scopes](https://noah-gao.gitbook.io/membase-user-guide/authentication); this page is the screen.

Connected apps (Claude, ChatGPT, Cursor…) do not need one: they connect through Connect and
approve on the consent screen.

## Where keys live

**Connect › Skills › Manage keys** opens the keys page. Every key is one row: name and hint,
access, reach, expiry, last use. **Active**, **Expired** and **Revoked** are the three
segments; a key that has stopped working stays listed for thirty days so it can be told
apart. **Create key** makes one; the name opens the key's page; **⋯** at the end of a row
holds **Rotate…** and **Revoke…**.

## Create a key

1. **Name** it after what will hold it, for example *nightly-notes script*, so a revocation stops one thing.
2. **Access.** *Read* (search and list), *Read & write* (also add a memory or a document), *Full access* (also delete a document or forget a memory, each only with `confirm=true`).
3. **Reach.** Nothing is ticked to begin with. Tick the Memories it may use, or **All memories, including ones you make later**. **Your profile** is the last row.
4. **Expires.** Never, 30 days, 90 days or a year.
5. **Create key.**

The token appears once, in full. Afterwards the app shows only its hint, such as
`mbk_7f3a92d1…c91e`, which tells keys apart and can never be used. The same screen gives the
token in six shapes: **Tell your AI** (the one sentence that installs the skill with this
key), a `curl` call, Python, TypeScript, the Claude Code command, and the `mcp.json` block
Cursor and VS Code read. **Send a test call** makes one real call with the new key from the
browser and reports *Verified* and how many Memories the key can see.

```bash
export MEMBASE_API_KEY="mbk_…"
```

## A key's page

* **Access.** Change the level with the segmented control; the tools it now holds are listed under it. Applies on the next call; the token stays the same.
* **Reach.** A switch per Memory, one for *All memories, including ones you make later*, one for *Your profile*. Off takes effect on the next call.
* **Expiry.** Cannot be extended; to change it, rotate.
* **Usage.** Calls in the last 30 or 7 days, how many were *refused*, the tool it calls most.
* **Recent activity.** The last calls one by one: tool, Memory, what came back, when. A refused call is red and says why.

Click the name to rename. Which Memories a key reaches can also be switched from the
Memory's own page under **Use in**, where keys are listed after the apps.

## Rotate and revoke

**Rotate…** makes a new key with the same access, reach and expiry, and asks what to do with
the current one. By default it keeps working until revoked, so the value can be swapped in
the environment first.

**Revoke…** ends a key at once: every script holding the token fails on its next call. The
dialog shows the hint and warns when the key was used in the last week. A revoked key stays
under **Revoked** for thirty days.

## From code

Keys are managed with the owner's session, never with another key:

| Call | Does |
|---|---|
| `POST /v1/api-keys` | mint: name, level, container list or `all_views`, profile tick, `expires_in_days` |
| `PATCH /v1/api-keys/{id}` | change name, level, profile tick, all-containers reach |
| `PATCH /v1/exposures/{id}/views` | change the container list |
| `GET /v1/api-keys/{id}/usage` | the Usage card as JSON |
| `DELETE /v1/exposures/{id}` | revoke |

## If something looks wrong

| It says | What it means | Do |
|---|---|---|
| `403 … may not use that container` | the key does not reach that Memory | switch it on under Reach |
| `403 … not in this agent's capability profile` | the call is above the key's access | raise the level |
| `"status": "confirmation_required"` | a delete or forget without `confirm=true` | say `confirm=true` after the person decided; only a key can |
| `422 … no model` | the memory cannot run a turn yet | [AI Setup](ai-setup.md) |
| *refused* in Recent activity | the key asked for something it does not have | the row says which Memory or verb; adjust Reach or Access |
| *Expired* on the row | the token's day has passed | **⋯ › Create a similar key** |
| *in 6 days* on the row (amber) | the key expires within a week | **Rotate…** and swap the value |
