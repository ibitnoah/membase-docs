---
title: FAQ
description: "Short answers to the questions people ask first: what a Memory is, when material is read, why the first search is slow, what an app can and cannot do, and what cannot be undone."
---

# FAQ

## What is a Memory?

One named part of your world, kept up to date: *Project decisions*, *Reading notes*,
*Customers*. It has an instruction (what it keeps), sources (what it reads), a schedule (how
often it re-reads) and a list of apps and keys that may use it. In the API it is a
**container**. [How Membase fits together](concepts.md).

## I added a folder. Why does my AI not know about it yet?

Adding a source never reads it. A Memory learns when you press **Run** (or **Update now**, the
same button while a source holds unread material), when its schedule fires, or when code calls
`add_document`. A source's sync only fetches raw material.
[Bring your material in](https://noah-gao.gitbook.io/membase-user-guide/use/getting-started/bring-material-in).

## Why does the first search take a minute?

A search runs inside your own agent container, against what your Memories know. An idle
container is stopped, and the first request wakes it. The next one is quick. The SDKs wait
90 seconds for this reason, and the answer marks any Memory that could not answer yet.
[How Membase works](https://noah-gao.gitbook.io/membase-user-guide/build/core/how-membase-works#a-search-is-a-turn).

## Where does my memory live? Does Membase keep a copy?

In your account's own container, on its own volume. The platform holds accounts, credentials,
bindings and billing; it transits raw bytes from sources and proxies model calls; it does not
store, index or project memory content. Your export is yours.

## What can a connected app do?

An app connected through Connect (Claude, ChatGPT, Cursor…) can list the Memories you switched
on for it and search them, and read your profile if you ticked **Let it know about you**. It
cannot add, delete or forget anything, and it cannot see a Memory that is not switched on.
An AI that needs to write gets a developer key instead, at the level you choose.
[Access control](https://noah-gao.gitbook.io/membase-user-guide/connect/reference/access-control).

## How do I stop an app?

Turn its switch off on the Memory's **Use in** card, or press **Disconnect…** on the app's
page in Connect. Both take effect on the app's very next question. Removing the connector
inside the app does not tell Membase, so do it on Connect to be sure.

## Which model does it use? Do I need my own key?

Every turn needs a model. Under **AI Setup** you bring a key for a provider (OpenAI, Anthropic,
Gemini, DeepSeek, Kimi, Qwen, OpenRouter or any OpenAI-compatible endpoint) or connect a Claude
or ChatGPT subscription. The credential never enters your container; the platform makes the
call. A free account has free turns to start with, and keeps its memory when they are spent.
[AI Setup](https://noah-gao.gitbook.io/membase-user-guide/use/features/ai-setup).

## Why is a bad key a 403 and not a 401?

Because the request was authenticated as *some* caller (the key id is in the token) and that
caller is not allowed. Only a request with no bearer at all is `401`.
[Authentication & Scopes](https://noah-gao.gitbook.io/membase-user-guide/build/core/authentication#what-a-wrong-credential-looks-like).

## My product serves many people. One key for all of them?

No. The account is the tenant, and a Memory is a topic, not an end user. Each person has their
own Membase account and your product reaches their memory with their own key or their own
consent. [Multi-user isolation](https://noah-gao.gitbook.io/membase-user-guide/build/core/multi-user-isolation).

## What is the profile?

The standing facts your assistant keeps about you: who you are and lasting preferences
(`static`), and what changed recently (`dynamic`). It is granted separately from reach because
it is about you, not about a topic. An AI should read it first in a new conversation.

## Can search filter on metadata?

No. Search takes a question, an optional Memory and a limit; there is no server-side index to
apply filters to. Put what matters in the content, and keep `custom_id` and `metadata` for your
own bookkeeping. [Memory operations](https://noah-gao.gitbook.io/membase-user-guide/build/core/memory-operations).

## What cannot be undone?

Deleting a Memory and what it learned, deleting a source's imported material, forgetting a
conversation, and deleting the account. Each asks first and says the irreversible part first.
Everything else (Disconnect, Revoke, Remove, End, Pause, Unsubscribe) can be redone later, and
takes effect at once. [How Membase fits together](concepts.md#stopping-and-undoing).

## How is this different from RAG over my files?

A Memory reads its sources in a learning turn and keeps what it learned as facts, with
provenance; a search answers from those facts, not from chunks of the transcript. The numbers
are on [Benchmarks](https://noah-gao.gitbook.io/membase-user-guide/build/core/benchmarks).

## When is Local Memory coming?

The engine that runs in the hosted container is the same one the local release will ship.
Documentation comes with that release; until then the landing page keeps a card for it.
