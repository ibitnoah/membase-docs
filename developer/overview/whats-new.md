---
title: What's new
description: "What changed in Membase, newest first: the SDKs, developer keys, the agent protocol, sources, sign-in and plans."
---

# What's new

Newest first. Each entry says what changed for a person using the app, an AI connected to it,
or code written against it.

## September 2026

**Guides organized around your task (28 Sep).** Start with a working Memory, choose a source
(including the new [Notion guide](https://noah-gao.gitbook.io/membase-user-guide/use/bring-your-material-in/notion)), or connect your AI.
Telegram is under using your assistant; Studio and Agents are under Advanced. The API
quickstart now waits for learning and checks search errors, with a separate
[troubleshooting guide](https://noah-gao.gitbook.io/membase-user-guide/build/reference/troubleshooting). Existing page addresses redirect to their
new locations.

**Python and TypeScript SDKs (24 Sep).** `pip install membase-sdk` and `npm install membase-sdk`:
one client over a developer key with `add`, `search`, `profile`, `ask` and the `containers`,
`documents` and `memories` resources under them. One error class per HTTP status, retries on
`429` and `5xx`, a 90-second default timeout for the first search after a quiet spell.
[Python and TypeScript SDKs](https://noah-gao.gitbook.io/membase-user-guide/build/reference/sdk-quickstart).

**Space is called Files (23 Sep).** The file manager is *Files* everywhere a person reads it,
and storage plans are *Storage Free / Standard / Pro*. Routes and connector ids are unchanged.

**Every sign-in provider is read (22 Sep).** Google and X sign-ins now land with an identity;
an account records which door it came through, and the default display name is email, then
handle, then wallet.

**Free plan dormancy (22 Sep).** A free account keeps its memory after its free turns are
spent, but cannot run a turn until it brings a model of its own under AI Setup or moves to a
paid plan. Code sees `422` with `reason: dormant` and the three ways back.

**Notion as a source (22 Sep).** Authorise once, pick the pages, and the selection becomes a
source a Memory reads on its next run. A sync fetches raw material only; nothing is read into
memory until the Memory runs.

**One new account per address per half hour (21 Sep).** Creating an account provisions its
container, so sign-up is throttled per client IP. Existing accounts sign in freely.

**Developer keys, and Connect in two tabs (17 Sep).** A key has a name and a hint, an access
level (Read, Read & write, Full access), a reach (which Memories, and the profile as its own
row), an expiry and a usage page. It renames, narrows or widens in place, rotates and revokes.
Connect's **MCP** tab is for an AI app (OAuth in the browser, read-only); its **Skills** tab is
one sentence for an AI that runs commands. [Connect your AI](https://noah-gao.gitbook.io/membase-user-guide/connect).

**The agent protocol (17 Sep).** One table of nine tools served over MCP, REST and the SDKs:
`list_containers`, `search_memories`, `get_profile`, `list_documents`, `get_document`,
`add_memory`, `add_document`, `delete_document`, `forget_memory`, plus `ask_agent` on a
credential that exposes an agent. The older `memory_*` names still answer.
[API reference](https://noah-gao.gitbook.io/membase-user-guide/build/reference/api-reference).

**Invitations (16 Sep).** Every account owns one invite code; a link carries it, and
attribution is decided once when the invitee's account is made.

## Earlier

**Sessions are venues (8 Sep).** A conversation on Home and the Telegram chat are two
conversations over one memory; ending or forgetting one does not touch the other.

**Sources hand over bytes, Memories read them (1 Sep).** A source sync never runs a model. The
Memory's run, pressed by hand, fired by a schedule or started by `add_document`, is the only
thing that reads material into memory.

**Memory lives in the account's container (August).** Each account has its own agent
container and volume; the platform keeps accounts, credentials and billing, transits raw bytes,
proxies model calls, and never stores or indexes memory content.
[How Membase works](https://noah-gao.gitbook.io/membase-user-guide/build/concepts/how-membase-works).
