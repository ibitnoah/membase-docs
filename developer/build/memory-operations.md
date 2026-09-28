---
title: Memory operations
description: "Every verb of the agent protocol end to end: containers, search, the profile, notes, documents, ask, forget and delete, what a run does to them, and what code sees when the app does something."
---

# Memory operations

The API has three nouns and a handful of verbs. This page walks them in the order a program
meets them, says what each does inside the user's account, and what code sees when the
person, a schedule or the app does something on its side. Every snippet is the Python SDK;
the TypeScript and REST shapes are on the [Python and TypeScript SDKs](sdk-quickstart.md) and the
[API reference](api-reference.md).

| Noun | In the app | What it is |
|---|---|---|
| **container** | a *Memory* | one named space of memory, with the agent that keeps it and the material it reads |
| **memory** | an item on a Memory's page | one fact the container holds, or one fact of the user's profile |
| **document** | a row under a Memory's sources | one piece of raw material a container read: a file, a note, a page |

## Containers

```python
containers = client.containers.list()["containers"]           # what the key reaches
decisions = next(c for c in containers if c["name"] == "Project decisions")
```

`list_containers` returns each Memory in the credential's reach as `{id, name, description, …}`;
the `id` is the `container` every other call takes. A Memory outside the reach is not listed
and cannot be told apart from one that does not exist. The assistant's own memory is always
the first tile in the app and, when reached, the first container here.

What a Memory's tile says in the app, and what it means to code:

| Tile word | Meaning | In the API |
|---|---|---|
| *Empty* | no instruction yet | listed, searches answer nothing |
| *Add a source* | reads nothing yet | listed, `list_documents` is empty |
| *On sale* | listed on the Marketplace | buyers hold agent-endpoint credentials to it |
| *Subscribed* | bought from someone else | `ask_agent` only; no documents, no writes |

Renaming a Memory changes `name` and `container_name`; the id never changes. Deleting a
Memory is the owner's, in the app, and has no API verb.

## Search

```python
hits = client.search("what did we decide about the ledger", container=decisions["id"], limit=5)
for h in hits["results"]:
    print(h["container_name"], "·", h["content"])
```

Omit `container` to search every Memory in reach at once; each hit still names its container.
The answer lists passages most relevant first, and `containers[]` marks any Memory that could
not answer yet.

A search is a **turn** inside the account's container, not a query on an index. The first one
after a quiet spell includes runtime startup; the SDKs default to 90
seconds per request, which is configurable and is not a latency guarantee. There are no metadata filters, because there is no server-side index to apply
them to: put what matters in the content. Search is retrieval; `ask` is an answer.

A Memory with no learned content may return no results. An explicitly requested container
outside the key’s reach is refused with `403`; an untargeted search omits it. Check
`containers[].error` before interpreting an empty result: an unavailable model is a failed
search, not evidence that no matching memory exists.

## The profile

```python
p = client.profile(q="working hours")
p["static"], p["dynamic"], p["results"]
```

The profile sits beside the containers: the standing facts the assistant keeps about the
person (`static`: who they are, lasting preferences) and the most recently changed ones
(`dynamic`). `get_profile` reads it, and only on a credential whose owner ticked the profile;
without the tick the operation is not offered at all. Read it once, near the start of a
conversation. Code writes to it with `add_memory` and `static=True`.

## Add a memory

```python
client.memories.add("We settled on Postgres for the ledger.", container=decisions["id"])   # a note it reads
client.memories.add("The user prefers dark mode.", static=True)                            # a standing fact → the profile
```

A note is raw material, and this call attempts to start its learning run asynchronously.
Use the returned `document_id` and the same polling as a document write; acceptance is not
completion. A standing fact about the person
goes to the profile instead of any Memory. With exactly one Memory in reach `container` may be
omitted; with several it is required. Needs a Read & write key.

## Add a document

```python
doc = client.add("…the text…", container="mv-…", title="Call with Acme",
                 custom_id="call-2026-09-24", metadata={"source": "crm"})
doc["status"]          # "queued"
doc["document_id"]     # poll documents.get(id)["learned"]
```

The text (or a public `url` to fetch, one or the other) lands in the Memory's upload folder in
the account's Files, is connected as a source on the spot, and a learning turn starts on its
own; the call answers `202`. The same `custom_id` again is a no-op, so retries are safe.
`metadata` is kept for your bookkeeping; search does not filter on it. `document_id` can be
`null` on the `202` when the folder sync had not written the row yet; list the container's
documents a moment later. Uploads are bounded by the account's plan (32 MiB per file on the
current plans).

### What the `202` becomes

When a learning run starts, it appears on the account’s Activity page. After a successful
run over that material, the document’s `learned` is true and search can retrieve what it
learned. A `202` alone does not guarantee that a run started: it can be deferred while the
agent is paused or capacity is unavailable. Inspect `learning_run` and the Memory’s Report. If it ends in *Run failed*,
the document is still listed and still unread, and the run's report says why; usually the
account has no working model under AI Setup, or a source needs reauthorization.

### Documents

```python
docs = client.documents.list(container="mv-…")["documents"]     # newest first
for d in docs:
    detail = client.documents.get(d["id"])
    print(d["id"], d.get("title"), "learned" if detail.get("learned") else "unread")

client.documents.get("srcitem_…")
client.documents.delete("srcitem_…", confirm=True)                # Full access
```

A document is one piece of raw material a Memory has read or will read: a file in a folder
source, an uploaded file, a Notion page, a conversation the browser extension captured, or
what `add_document` handed in. `learned` is false until a run has committed it; it is the flag
to poll after `add_document`.

### Sync and run are two different things

A **sync** fetches raw material and stops. It runs no model, reads nothing into memory and
never changes what a search answers. A **run** (the learning turn) is what reads the unread
material. In the app the Memory's button says **Update now** while a source holds material
it has not read; a schedule does the same on a cadence; `add_document` starts a run itself.
"Unread" is decided by versions, not clocks: a sync that changed nothing, or a run on another
Memory, cannot make this one look up to date.

`add_document` normally starts learning without a schedule; if it is deferred, use the
Memory’s next run after fixing the cause. A schedule
matters for material that arrives without a call: a folder the person keeps adding to, a
Notion selection that changes. When a schedule fires, nothing changes in the API except the
memory itself: `learned` turns true and the next search answers from what the run learned.
There is no API to create or pause a schedule; it is the owner's, in the app.

## Ask

```python
answer = client.ask("Summarise what the seller learned about Postgres RLS.")
```

`ask_agent` is the other kind of read next to search: an agent's answer from what its Memory
learned. It answers only on a credential that **exposes** an agent, a marketplace
subscription or a share the owner made; an ordinary developer key holds no agent, and the
call is refused with `403 · unauthorized · tool 'agent_invoke' is not in this agent's
capability profile`. The buyer gets answers, never files. A product that wants to put a model
over the person's memory uses search and its own model ([Claude API](claude-api.md),
[OpenAI API](openai-api.md)); a product that wants a Memory's own agent to answer subscribes
to it.

`workflow_invoke` belongs to workflow exposures, a canvas run as an endpoint, and is offered
only over MCP on a credential that exposes one.

## Forget and delete

```python
client.memories.forget("12", container=decisions["id"], confirm=True)
client.documents.delete("srcitem_…", confirm=True)
```

Both need a Full access key, and even then they do not run on a bare call: without
`confirm=true` the answer is `200` with `status: confirmation_required` and a `how` sentence
to relay to the person. Pass `confirm=true` only after they agreed; from a developer key that
counts as the owner's confirmation. A consent-minted client can never confirm. What is
forgotten or deleted is gone from the next search; nothing on the platform caches it.

Deleting a Memory, a source's imported material or the account has no API verb.

## Rules

```python
client.rules()
```

`memory_rules` returns the user's standing rules for this credential: what the skill tells an
AI about behaving beneath the ceiling the server enforces.

## What code sees when the app does something

| The person | Code sees |
|---|---|
| presses **Run** / **Update now**, or a schedule fires | documents' `learned` turns true; the next search answers from the run |
| switches a Memory off for the key under **Use in** | `403 · may not use that container` on the very next call, same token |
| lowers the key's access | `403 · not in this agent's capability profile` on the verbs above it |
| revokes the key, or it expires | `403 · unauthorized` on every call; mint another |
| removes a source | its documents stop being listed; what was learned stays until the Memory is deleted |
| deletes the Memory | it is no longer listed; searches no longer answer from it |
| has no working model for its agent | a failed turn, surfaced as `422 · capability_unavailable` or a search entry in `containers[].error`; stored memory remains, documents may wait unread |
| is on the free plan with its free turns spent | `422 · reason: dormant` until they bring a model or move to a paid plan |
| exceeds the concurrent-turn budget | `429 · rate_limited`; the SDK retries twice |

## Rules of the road

* **Profile once, search per question.** The profile is small and standing; search is a turn inside the user's container.
* **Cite the container.** Every hit names `container_name`; say where an answer came from.
* **Save what the user supplied,** in their words, and only when they asked or plainly meant to.
* **Never confirm on your own.** A delete or forget without `confirm=true` answers with a `how` sentence; relay it and stop.
* **Treat `403` as withdrawn access.** The owner narrowed or revoked the key; do not retry with it.
* **Allow for cold starts.** Configure a suitable timeout and inspect per-Memory errors; see [API troubleshooting](troubleshooting.md).
* **Do not store to filter later.** There are no server-side metadata filters; use `custom_id` and `metadata` for your own bookkeeping.
* **The token stays out of logs.** Log the key's hint (`mbk_7f3a92d1…`), never the token.
