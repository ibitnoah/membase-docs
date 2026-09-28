---
title: FAQ
description: "Short answers to the questions people ask first: what a Memory is, when material is read, why the first search is slow, what an app can and cannot do, and what cannot be undone."
---

# FAQ

## What is a Memory?

One named part of your world, kept up to date: *Project decisions*, *Reading notes*,
*Customers*. It has an instruction (what it keeps), sources (what it reads), a schedule (how
often it re-reads) and a list of apps and keys that may use it. [How Membase fits together](concepts.md).

## I added a folder. Why does my AI not know about it yet?

Adding a source never reads it. A Memory learns when you press **Run** (or **Update now**, the
same button while a source holds unread material), when its schedule fires, or when code calls
`add_document`. A source's sync only fetches raw material.
[Bring your material in](https://noah-gao.gitbook.io/membase-user-guide/use/bring-your-material-in/bring-material-in).

## Why does the first search take a minute?

Your account’s memory may need to wake after it has been idle. Searching also runs a model
to find relevant material. Allow time for that first request; if it fails, check the Memory’s
Report and model settings instead of assuming the Memory is empty.
[How Membase works](https://noah-gao.gitbook.io/membase-user-guide/build/concepts/how-membase-works#a-search-is-a-turn).

## Where does my memory live? Does Membase keep a copy?

In your account's own container, on its own volume. The platform holds accounts, credentials,
bindings and billing; it transits raw bytes from sources and proxies model calls; it does not
store, index or project memory content. Your export is yours.

## What can a connected app do?

An app connected through Connect (Claude, ChatGPT, Cursor…) can list the Memories you switched
on for it and search them, and read your profile if you ticked **Let it know about you**. It
cannot add, delete or forget anything, and it cannot see a Memory that is not switched on.
An AI that needs to write gets a developer key instead, at the level you choose.
[Access control](https://noah-gao.gitbook.io/membase-user-guide/connect/manage-access/access-control).

## How do I stop an app?

Turn its switch off on the Memory's **Use in** card, or press **Disconnect…** on the app's
page in Connect. Both take effect on the app's very next question. Removing the connector
inside the app does not tell Membase, so do it on Connect to be sure.

## Which model does it use? Do I need my own key?

Every turn needs a model. Under **AI Setup** you bring a key for a provider (OpenAI, Anthropic,
Gemini, DeepSeek, Kimi, Qwen, OpenRouter or any OpenAI-compatible endpoint) or connect a Claude
or ChatGPT subscription. The credential never enters your container; the platform makes the
call. A free account has free turns to start with, and keeps its memory when they are spent.
[AI Setup](https://noah-gao.gitbook.io/membase-user-guide/use/account-and-models/ai-setup).

## What is the profile?

The standing facts your assistant keeps about you: who you are and lasting preferences
and what changed recently. It is granted separately from reach because
it is about you, not about a topic. An AI should read it first in a new conversation.

## What cannot be undone?

Deleting a Memory and what it learned, deleting a source's imported material, forgetting a
conversation, and deleting the account. Each asks first and says the irreversible part first.
Everything else (Disconnect, Revoke, Remove, End, Pause, Unsubscribe) can be redone later, and
takes effect at once. [How Membase fits together](concepts.md#stopping-and-undoing).

## How is this different from RAG over my files?

A Memory reads its sources in a learning turn and keeps what it learned as facts, with
provenance; a search answers from those facts, not from chunks of the transcript. The numbers
are on [Benchmarks](benchmarks.md).

## When is Local Memory coming?

This guide covers the hosted app. Local Memory setup is not documented here yet; check
[What’s new](whats-new.md) for release announcements rather than using hosted setup steps
as local installation instructions.

Questions about API status codes, metadata filters or serving multiple users belong in
[API troubleshooting](https://noah-gao.gitbook.io/membase-user-guide/build/reference/troubleshooting).
