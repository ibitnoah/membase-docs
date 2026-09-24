---
description: How the hosted memory layer is built, and what that means for code that uses it.
---

# Memory Platform

The Memory Platform is the hosted Membase at `https://www.app.membase.io`. A person hands it
material once; it keeps a living memory of that material; every AI they connect, and every
key they mint, reads the same memory. This page is the model behind the API: what the nouns
are, where the bytes live, who may do what, and what the platform deliberately does not do.

## Where memory lives

Every account has its **own agent container**, a small VM with its own volume. That volume is
the memory of record: the account's files, the documents its Memories read, and the memory
each Memory has learned. The platform holds control-plane data (accounts, credentials,
bindings, billing) and transits raw bytes; it does not index, store or hand-maintain memory
content.

Three consequences for code:

* **A search is a turn.** `search_memories` runs inside the account's container. The first call after a quiet spell can take up to a minute while the container wakes; the answer marks in `containers[]` any container that could not answer yet.
* **There is no server-side content index to filter.** Search takes `q`, an optional `container` and `limit`, nothing else. Unknown parameters are refused, never silently ignored.
* **Deletion wins.** Revoking a credential, narrowing its reach or deleting a document takes effect at the next call, ahead of any cache.

## The three nouns

| API | In the app | What it is |
|---|---|---|
| **container** | a *Memory* | one named space of memory, with the agent that keeps it and the material it reads |
| **memory** | an entry on the Memory page | one fact the container holds, or one fact of the user's profile |
| **document** | a row under the Memory's sources | one piece of raw material a container read: a file, a note, a page |

Plain verbs: `list`, `search`, `get`, `add`, `delete`, `forget`, `ask`. The
[API Reference](api-reference.md) has every operation.

A **profile** sits beside the containers: the standing facts the assistant keeps about the
user (`static`) and the most recently changed ones (`dynamic`). `get_profile` reads it;
`add_memory` with `static=true` writes to it.

## How material becomes memory

```
Files folder · Notion · Unibase Memory · add_document  ──►  the Memory's documents
                                                              │  learning turn (Run / Update now / schedule)
                                                              ▼
                                                         what the Memory knows  ──►  search_memories · MCP · the assistant
```

A Memory reads **sources**: a folder in the account's Files, a Notion selection, the Unibase
Memory browser extension, or whatever `add_document` hands it. Nothing is read until a
learning turn runs; the person presses **Run** (or **Update now** when a source holds unread
material), or sets a schedule. `add_document` starts that turn on its own and answers `202`;
`learned` on the document turns true when the turn has committed.

A trusted run commits its non-destructive output automatically, provenance-tagged and
reversible. Destructive changes (delete, forget) always need explicit confirmation.

## Credentials

Every call carries a bearer. Two kinds exist, and they differ in one thing: who holds them.

| | Consent token | Developer key |
|---|---|---|
| Minted by | the user approving an app on the OAuth consent screen | the user, under Connect › Developer keys |
| Held by | Claude, ChatGPT, Cursor, Codex… | the user's own code, or an AI running the skill |
| Access | `list_containers`, `search_memories`, `get_profile` when ticked | Read, Read & write or Full access |
| Reach | the Memories ticked on consent | the Memories ticked on the key, or all including later ones |
| Can confirm a removal | never | yes, with `confirm=true` |
| Token | opaque | `mbk_<key id>_<token>`, shown once; the key keeps a hint |

Both are bindings on the account. Turning a Memory off on the Connect page narrows every
credential at its next call; revoking a credential ends it at once, and it stays listed as
revoked for thirty days. A withheld container is refused the same way as one that does not
exist.

### Access levels

| Level | Adds | Grant |
|---|---|---|
| **Read** | list containers, search, profile (when ticked), list and get documents, rules | `read` |
| **Read & write** | add a memory, add a document | `suggest` |
| **Full access** | delete a document, forget a memory, each only with `confirm=true` | `manage` |

`ask_agent` and `workflow_invoke` belong to agent and workflow exposures (a Memory sold or
shared as an endpoint), not to keys.

### Managing keys from code

| Call | Does |
|---|---|
| `POST /v1/api-keys` | mint a key |
| `PATCH /v1/api-keys/{id}` | change its name, level, profile tick or all-containers reach; the next call reads the new grant |
| `PATCH /v1/exposures/{id}/views` | change its container list |
| `GET /v1/api-keys/{id}/usage` | calls per day, refusals, per tool, the last twenty |
| `DELETE /v1/exposures/{id}` | revoke it, at once |

## One account per person

The account is the tenant. A container is a topic, not an end user. A product that serves
many people gives each of them their own Membase account, and reads their memory with their
own key or their own consent; there are no application-wide keys and no sub-tenant tags.

## Limits and errors

| Status | Code | When |
|---|---|---|
| 400 | `validation` | a missing `q`, both `content` and `url`, an ambiguous `container` |
| 403 | `unauthorized` | outside the credential's reach, or a verb above its level |
| 404 | `not_found` | an unknown document or memory id |
| 422 | `capability_unavailable` | the account's memory cannot run a turn here (no agent container, no model) |
| 429 | `rate_limited` | the account's concurrent-turn budget; retry later |

Uploads through `add_document` are bounded by the account's plan (32 MiB per file on the
current plans). An account on the free plan whose free turns are spent keeps its memory but
cannot run turns until it brings a model of its own under AI Setup or moves to a paid plan;
reads then answer `422` with `reason: dormant`.

## Marketplace

A Memory can be sold as an **agent endpoint**: buyers subscribe, receive a credential of their
own and call `ask_agent` against the seller's agent, which answers from what it learned. The
seller's memory never leaves their container; the buyer gets answers, not files. Subscriptions
are per period, and lapsing stops metering at once.

## The app around it

The **User Guide** documents the product screens: Home (the conversation with the assistant),
Memory, Studio, Agents, Schedules, Files, Activity, AI Setup, Connect, Marketplace and
Settings, one page per screen, with the status words each shows and what to do about them.
