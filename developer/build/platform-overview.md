---
title: Platform overview
description: "The hosted Membase at app.membase.io as your code meets it: the three nouns, what each screen of the app is to the API, credentials, one account per person, limits, the Marketplace, and what can be undone."
---

# Platform overview

The Memory Platform is the hosted Membase at `https://www.app.membase.io`. A person hands it
material once; it keeps a living memory of that material; every AI they connect, and every
key they mint, reads the same memory. The mechanism is on
[How Membase works](how-membase-works.md); this page is the product around it, written for
someone whose code will meet what the person sees. The screens themselves, one page each,
are in the **Use Membase** tab.

## The three nouns

| API | In the app | What it is |
|---|---|---|
| **container** | a *Memory* | one named space of memory, with the agent that keeps it and the material it reads |
| **memory** | an item on a Memory's page | one fact the container holds, or one fact of the user's profile |
| **document** | a row under a Memory's sources | one piece of raw material a container read: a file, a note, a page |

Plain verbs: `list`, `search`, `get`, `add`, `delete`, `forget`, `ask`.
[Memory operations](memory-operations.md) walks them; the [API reference](api-reference.md)
has every operation.

A **profile** sits beside the containers: the standing facts the assistant keeps about the
user (`static`) and the most recently changed ones (`dynamic`). `get_profile` reads it;
`add_memory` with `static=true` writes to it.

## The screens

| Screen | The person uses it to | Your code meets it as |
|---|---|---|
| [Home](https://noah-gao.gitbook.io/membase-user-guide/use/features/home) | talk to the assistant, the one agent that is theirs by default; reach it on Telegram | the profile it keeps, and the assistant's own memory, always the first container |
| [Memory](https://noah-gao.gitbook.io/membase-user-guide/use/features/memory) | make, run and inspect Memories | `list_containers`, `search_memories`, the items `forget_memory` removes; `learned` turning true after a run |
| [Files](https://noah-gao.gitbook.io/membase-user-guide/use/features/files) | keep files, and hand folders, uploads, Notion pages and captured conversations to a Memory | `add_document`, `list_documents`, `get_document`, `delete_document` |
| [Studio](https://noah-gao.gitbook.io/membase-user-guide/use/features/studio) | edit a Memory's pipeline canvas | nothing; a canvas run as an endpoint is `workflow_invoke` over MCP |
| [Agents](https://noah-gao.gitbook.io/membase-user-guide/use/features/agents) | build agents beyond the assistant | `ask_agent` on an agent-endpoint credential |
| [Schedules](https://noah-gao.gitbook.io/membase-user-guide/use/features/schedules) | run Memories on a cadence | `learned` turning true without a call of yours |
| [Activity](https://noah-gao.gitbook.io/membase-user-guide/use/features/activity) | see what ran | the run a `202` from `add_document` started; a key's own calls are on the key's page |
| [AI Setup](https://noah-gao.gitbook.io/membase-user-guide/use/features/ai-setup) | choose the account's model | `422 · capability_unavailable` when there is none |
| [Connect](https://noah-gao.gitbook.io/membase-user-guide/use/features/connect) | let an AI app read Memories, mint keys | consent tokens and developer keys: [Authentication & Scopes](authentication.md) |
| [Marketplace](https://noah-gao.gitbook.io/membase-user-guide/use/features/marketplace) | sell and buy access to a Memory | subscriptions, `ask_agent` |
| [Settings](https://noah-gao.gitbook.io/membase-user-guide/use/features/settings) | plan, export, delete | `422 · reason: dormant` on a free plan whose turns are spent |

## Credentials

Every call carries a bearer: a **developer key** the owner minted, or a **consent token** an
app received on the OAuth consent screen. A key has a level (Read, Read & write, Full
access), a reach (the Memories it may use) and an expiry; a consent token is read-only and
reaches what the user ticked. Both are bindings on the account and change live on the
Connect page. [Authentication & Scopes](authentication.md) has the whole model; the keys
screen is under [Connect](https://noah-gao.gitbook.io/membase-user-guide/use/features/connect#developer-keys) in the user guide.

A key's calls are on the key's page, not on Activity: **Usage** (calls in the last 30 or 7
days, how many were *refused*, the tool it calls most) and **Recent activity** (each call:
tool, Memory, what came back, when; a refused call in red with the reason). The same numbers
are on `GET /v1/api-keys/{id}/usage` with the owner's session.

## One account per person

The account is the tenant. A container is a topic, not an end user. A product that serves
many people gives each of them their own Membase account, and reads their memory with their
own key or their own consent; there are no application-wide keys and no sub-tenant tags.
[Multi-user isolation](multi-user-isolation.md) has the patterns.

## Models

Every learning turn, every search and every assistant answer needs a model. The account
chooses it under AI Setup: a key of its own for a provider, or a Claude or ChatGPT
subscription, verified with one short real request before it is used. The credential never
enters the account's container; the platform makes the call. An account with no model keeps
its memory: searches over what it already learned still answer, and documents wait unread.
[How Membase works](how-membase-works.md#models).

## Limits and errors

| Status | Code | When |
|---|---|---|
| 400 | `validation` | a missing `q`, both `content` and `url`, an ambiguous `container` |
| 403 | `unauthorized` | outside the credential's reach, or a verb above its level |
| 404 | `not_found` | an unknown document or memory id |
| 422 | `capability_unavailable` | the account's memory cannot run a turn here (no agent container, no model) |
| 422 | `reason: dormant` | a free-plan account whose free turns are spent |
| 429 | `rate_limited` | the account's concurrent-turn budget; retry later |

Uploads through `add_document` are bounded by the account's plan (32 MiB per file on the
current plans). An account on the free plan whose free turns are spent keeps its memory but
cannot run turns until it brings a model of its own under AI Setup or moves to a paid plan.

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
Memory someone bought. The person's side of the same table is on
[How Membase fits together](https://noah-gao.gitbook.io/membase-user-guide/concepts#stopping-and-undoing).
