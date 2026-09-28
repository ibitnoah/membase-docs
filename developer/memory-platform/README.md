---
description: The hosted Membase at app.membase.io, one screen per page, and what each screen means for code that reads the same memory.
---

# Platform Overview

The Memory Platform is the hosted Membase at `https://www.app.membase.io`. A person hands it
material once; it keeps a living memory of that material; every AI they connect, and every
key they mint, reads the same memory. The mechanism is on
[How Membase Works](https://noah-gao.gitbook.io/membase-user-guide/how-membase-works); this section is the product around it, one
screen per page, written for someone whose code will meet what the person sees.

## The three nouns

| API | In the app | What it is |
|---|---|---|
| **container** | a *Memory* | one named space of memory, with the agent that keeps it and the material it reads |
| **memory** | an item on a Memory's page | one fact the container holds, or one fact of the user's profile |
| **document** | a row under a Memory's sources | one piece of raw material a container read: a file, a note, a page |

Plain verbs: `list`, `search`, `get`, `add`, `delete`, `forget`, `ask`. The
[API Reference](https://noah-gao.gitbook.io/membase-user-guide/api-reference) has every operation.

A **profile** sits beside the containers: the standing facts the assistant keeps about the
user (`static`) and the most recently changed ones (`dynamic`). `get_profile` reads it;
`add_memory` with `static=true` writes to it.

## The screens

| Screen | The person uses it to | Your code meets it as |
|---|---|---|
| **Home** | talk to the assistant, the one agent that is theirs by default | the profile it keeps, and the assistant's own memory, always the first container |
| [Memory](memories.md) | make, run and inspect Memories | `list_containers`, `search_memories`, the items `forget_memory` removes |
| [Files, and sources](sources-and-files.md) | keep files, and hand folders, uploads, Notion pages and captured conversations to a Memory | `add_document`, `list_documents`, `get_document`, `delete_document` |
| [Agents](agents.md) | build agents beyond the assistant | `ask_agent` on an agent-endpoint credential |
| [Schedules](schedules.md) | run Memories on a cadence | when `learned` turns true without a call of yours |
| [Activity](activity.md) | see what ran | what a key did, on the key's usage page |
| [AI Setup](ai-setup.md) | choose the account's model | `422 · capability_unavailable` when there is none |
| **Connect** | let an AI app read Memories, mint keys | consent tokens, [developer keys](developer-keys.md) |
| **Marketplace** | sell and buy access to a Memory | subscriptions, `ask_agent` |

## Credentials

Every call carries a bearer: a **developer key** the owner minted, or a **consent token** an
app received on the OAuth consent screen. A key has a level (Read, Read & write, Full
access), a reach (the Memories it may use) and an expiry; a consent token is read-only and
reaches what the user ticked. Both are bindings on the account and change live on the
Connect page. [Authentication & Scopes](https://noah-gao.gitbook.io/membase-user-guide/authentication) has the whole model;
[Developer Keys](developer-keys.md) has the screen.

## One account per person

The account is the tenant. A container is a topic, not an end user. A product that serves
many people gives each of them their own Membase account, and reads their memory with their
own key or their own consent; there are no application-wide keys and no sub-tenant tags.
[Multi-user Isolation](https://noah-gao.gitbook.io/membase-user-guide/multi-user-isolation) has the patterns.

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
are per period, and lapsing stops metering at once. Skills, packaged abilities for agents,
are the other half of the market.

## Stopping and undoing

Everything takes effect at once; nothing waits for a cache, a sync or a paid period.
Reversible: switching an app or key off a Memory, disconnecting an app, removing a source,
pausing a schedule, unsubscribing. Irreversible, and always confirmed first: deleting a
Memory and what it learned, deleting a source's imported material, forgetting a conversation,
deleting the account. Deletion wins over caches, projections, issued credentials, and a
Memory someone bought.
