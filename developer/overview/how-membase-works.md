---
description: Where memory lives, how material becomes memory, why a search is a turn, and what the platform deliberately does not do.
---

# How Membase Works

Membase keeps one living memory per person and lets every AI they use read it. This page is
the mechanism behind that sentence: where the bytes are, what turns them into memory, and
what a call does when it arrives.

## One container per account

Every account has its **own agent container**: a small VM with its own volume. That volume is
the memory of record. It holds the account's Files, the documents each Memory has read, and
what each Memory has learned. The assistant the person talks to on Home runs there too.

The platform around the container is a control plane. It holds accounts, credentials,
bindings, schedules and billing; it transits raw bytes from sources into the container; it
proxies model calls. It does not index memory content, does not keep a copy of it, and does
not let anyone hand-maintain it. If the container cannot confine something, the operation
fails rather than falling back to the platform.

```
        sources                       the account's container                  readers
 Files · Upload · Notion   ──►  documents  ──►  learning turn  ──►  memory   ──►  search_memories
 Unibase Memory · add_document        │                                 │          MCP clients
                                      │                                 │          the assistant
                                 raw bytes only                   what the Memory knows
```

## Material becomes memory in a learning turn

A **Memory** (a *container* in the API) is one named space of memory with an instruction
(what it keeps), sources (what it reads) and a schedule (how often). A **source** is where raw
material comes from: a folder in the account's Files, an upload, a Notion selection, the
Unibase Memory browser extension, or whatever `add_document` hands in.

Handing material in never reads it. A source sync lands raw bytes in the container's inbox
and stops; no model is involved. The Memory reads its unread material in a **learning turn**:
its agent works through the documents under the instruction and commits what it learned as
facts, provenance-tagged and reversible. A turn starts when the person presses **Run** (or
**Update now**, the same button while unread material waits), when the schedule fires, or
when `add_document` is called, which starts one on its own and answers `202`. The document's
`learned` flag turns true when the turn has committed.

"Unread" is decided by versions, not clocks: a sync that changed nothing, or a run on another
Memory, does not make this one look up to date.

A trusted run commits its non-destructive output automatically. Anything destructive, such as
deleting a document or forgetting a fact, always needs an explicit confirmation and is never
auto-committed.

## A search is a turn

`search_memories` does not query an index on the platform; it runs inside the account's
container against what its Memories know. Three things follow:

* **The first call after a quiet spell is slow.** An idle container is stopped, and the first request wakes it. Expect up to a minute; the SDKs default to a 90-second timeout for this reason. `containers[]` in the answer marks any container that could not answer yet.
* **There is nothing to filter on but the question.** Search takes `q`, an optional `container` and a `limit`. There are no metadata filters, because there is no server-side index to apply them to. Put what matters in the content.
* **Deletion wins.** A document deleted, a fact forgotten or a credential narrowed is gone on the next call; there is no cache in front of the container to outlive it.

Search is retrieval: passages, most relevant first, each naming its container. `ask_agent`
is the other kind of read, an agent's answer, and exists only on credentials that expose an
agent.

## The profile

Beside the Memories sits the **profile**: the standing facts the assistant keeps about the
person. `static` holds who they are and lasting preferences; `dynamic` holds what changed
recently. The assistant writes it from conversations; code writes it with `add_memory` and
`static=true`; `get_profile` reads it, and only on a credential whose owner ticked the
profile. It is the first thing an AI should read in a new conversation, and it is separate
from reach because it is about the person, not about a topic.

## Models

A learning turn, a search and the assistant's answers all need a model. The account chooses
it under **AI Setup**: a key of its own for a provider, or a Claude or ChatGPT subscription.
Every model call goes through the platform's inference service under the account's policy;
the container never holds a model credential. An account with no working model keeps its
memory but cannot run a turn, and reads answer `422` with `code: capability_unavailable`
until one is set.

## Credentials are bindings

A developer key or an app's consent is a **binding** on the account: a record of who may
reach which Memories at what level. The token identifies the binding; the binding's current
state decides the call. That is why a key can be narrowed, widened or revoked on the Connect
page without re-minting, and why the change applies on the very next request.
[Authentication & Scopes](authentication.md) has the levels and the reach rules.

## What the platform does not do

* It does not store, index or project memory content. The container is the only copy the platform serves from; the person's own backup is theirs.
* It does not run a model on a sync. Sources move bytes; only a learning turn reads them.
* It does not auto-commit anything destructive, from any caller.
* It does not hold application-wide keys or sub-tenants. One account is one person; see [Multi-user Isolation](multi-user-isolation.md).
* It does not let a model credential into a container, and it does not serve a subscription-backed turn from anywhere but its own proxy.

## Where to go next

| To | Read |
|---|---|
| make a first call | [API Quickstart](api-quickstart.md) |
| see every operation | [API Reference](api-reference.md) |
| understand the screens the person uses | [Platform Overview](https://noah-gao.gitbook.io/membase-user-guide/memory-platform) |
| connect an AI app | [Plugins Overview](https://noah-gao.gitbook.io/membase-user-guide/plugins-mcp) |
| see what the engine scores | [Benchmarks](benchmarks.md) |
