---
description: Files is the account's folder tree; a source is what a Memory reads from it, from Notion, from the browser extension or from add_document. Raw bytes in, documents out, nothing read until a run.
---

# Sources & Files

A Memory learns from what it is handed. Every way in ends the same way: the material lands in
the Memory's Add card as a **source**, becomes **documents** the Memory can read, and nothing
is read until the Memory runs.

| Way in | Good for | Where |
|---|---|---|
| A folder in Files | a project, a vault, anything already on disk | Memory page › Add card › **Your Files › Choose** |
| Upload | a handful of files | Memory page › Add card › **Upload Files › Upload** |
| Notion | a selection of pages from a workspace | source catalog › Notion |
| Unibase Memory | the person's chats with other assistants | Memory page › Add card › **Unibase Memory**; [Browser Extension](https://noah-gao.gitbook.io/membase-user-guide/plugins-mcp/browser-extension) |
| `add_document` | what your code produced | the API, with a Read & write key |
| Talking to the assistant | what the person tells it | Home or Telegram; the assistant's own memory only |

## Files

**Files** is the account's files: a real folder tree the assistant shares with the person.
Any folder in it can become a source. The locked system folders *Generated*, *Assets* and
*Trash* sit beside the person's own; deleted files go to *Trash* first. Browsing never wakes
the memory; only a Memory's run reads files.

A folder source is read **in place**: whatever the person later adds to the folder is picked
up on the Memory's next run, and there is nothing to sync. Making a folder a source is done
from the Memory (**Choose** on *Your Files*), not from Files.

An **upload** lands in a folder named after the Memory at the top of Files, connected as a
source on the spot; that folder follows the Memory when it is renamed. Markdown, text and
the common document formats are accepted, up to the plan's size per file (32 MiB on the
current plans).

## Documents

A **document** is one piece of raw material a Memory has read or will read: a file in a
folder source, an uploaded file, a Notion page, a captured conversation, or what
`add_document` handed in.

```python
docs = client.documents.list(container="mv-…")["documents"]     # newest first
for d in docs:
    print(d["id"], d.get("title"), "learned" if d.get("learned") else "unread")

client.documents.get("srcitem_…")
client.documents.delete("srcitem_…", confirm=True)                # Full access
```

`learned` is false until a run has committed the document; it is the flag to poll after
`add_document`. Deleting a document is destructive: it needs a Full access key, answers
`status: confirmation_required` without `confirm`, and is gone from the next search.

## `add_document`

```python
doc = client.add("…the text…", container="mv-…", title="Call with Acme",
                 custom_id="call-2026-09-24", metadata={"source": "crm"})
doc["status"]          # "queued"
doc["document_id"]     # poll documents.get(id)["learned"]
```

The text (or a public `url` to fetch, one or the other) lands in the Memory's upload folder
and a learning turn starts on its own; the call answers `202`. The same `custom_id` again is a
no-op, so retries are safe. `metadata` is kept for your bookkeeping; search does not filter
on it. `document_id` can be `null` on the `202` when the folder sync had not written the row
yet; list the container's documents a moment later.

## Sync and run are two different things

A **sync** fetches raw material and stops. It runs no model, reads nothing into memory and
never changes what a search answers. A **run** (the learning turn) is what reads the unread
material. The Memory's button says **Update now** while a source holds material it has not
read; a schedule does the same on a cadence; `add_document` starts a run itself.

Each source has its own page, with its own status:

| It says | What it means | Do |
|---|---|---|
| *Queued* / *Syncing…* | fetching raw material | wait; the Memory reads it on its next run |
| *Sync failed* | the last fetch failed | **Sync now**; if it repeats, the folder may have moved |
| *Needs reauthorization* | the source's own credential expired | **Reauthorize** |
| *Access revoked* / *Disconnected* | the source no longer grants access | **Reconnect** or remove it |

A folder source shows **Disconnect…** as its one verb, because a folder read in place has
nothing to sync. A connector source (Notion, Unibase Memory) shows a status word here, and
**Sync now**, **Reauthorize** or **Reconnect** when they apply.

## Removing

**Remove** on a source row stops the Memory reading it; the material stays in Files and can
be re-added. Deleting a source's imported material from the source page is irreversible and
confirmed first. Deleting the Memory takes everything it learned with it, whatever the
sources were.
