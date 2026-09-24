---
description: Get an API key, install the SDK, write a memory and search it. Five minutes to your first call.
---

# API Quickstart

You need a Membase account with at least one **Memory** (a *container* in the API). Sign in at
`https://www.app.membase.io`; the first-run guide makes the first Memory with you. Everything
below runs against `https://api.app.membase.io`.

## 1. Get an API key

In the app: **Connect › Developer keys › Create key**.

1. **Name** it after what will hold it, for example *nightly-notes script*.
2. Pick the **Access**. The level decides which operations the key gets:
   * **Read**: list, search, profile, documents.
   * **Read & write**: also add a memory or a document.
   * **Full access**: also delete a document or forget a memory. Each of those calls must still say `confirm=true`.
3. Pick the **Reach**: tick the Memories it may use, or *All memories, including ones you make later*. Tick **Your profile** if it may read the standing facts your assistant keeps about you.
4. Pick when it **Expires**, then **Create key**.

The token (`mbk_…`) is shown once. Copy it now; the key's page keeps only a hint afterwards,
plus its usage and recent calls. A lost key is rotated: mint a new one, revoke the old.

```bash
export MEMBASE_API_KEY="mbk_…"
```

For this quickstart pick **Read & write** and tick one Memory.

## 2. Install the SDK

{% tabs %}
{% tab title="Python" %}
```bash
pip install membase-sdk
```

```python
from membase_sdk import Membase
mb = Membase()          # reads MEMBASE_API_KEY
```
{% endtab %}

{% tab title="TypeScript" %}
```bash
npm install @membase/sdk
```

```ts
import { Membase } from "@membase/sdk";
const mb = new Membase();   // reads MEMBASE_API_KEY
```
{% endtab %}

{% tab title="curl" %}
No install. Two variables:

```bash
export MEMBASE_API_KEY="mbk_…"
export BASE="https://api.app.membase.io"
```
{% endtab %}
{% endtabs %}

The SDK is a thin client over the REST API: one method per operation, the same names, the same
access and reach rules (they live server-side and cannot be bypassed from code). The
[SDK Quickstart](sdk-quickstart.md) has every method.

## 3. Write a memory

{% tabs %}
{% tab title="Python" %}
```python
mb.add_memory("The user prefers dark mode.", static=True)      # a standing fact → the profile
mb.add_memory("We settled on Postgres for the ledger.")         # a note → the one Memory in reach
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
await mb.addMemory({ content: "The user prefers dark mode.", static: true });
await mb.addMemory({ content: "We settled on Postgres for the ledger." });
```
{% endtab %}

{% tab title="curl" %}
```bash
curl -s "$BASE/v1/memories" -H "Authorization: Bearer $MEMBASE_API_KEY" \
  -H 'content-type: application/json' \
  -d '{"content": "The user prefers dark mode.", "static": true}'

curl -s "$BASE/v1/memories" -H "Authorization: Bearer $MEMBASE_API_KEY" \
  -H 'content-type: application/json' \
  -d '{"content": "We settled on Postgres for the ledger."}'
```
{% endtab %}
{% endtabs %}

`static=true` is for standing facts about the user (name, role, lasting preferences); they go
to the profile. Everything else is a note the Memory reads. With exactly one Memory in reach
the `container` field may be omitted; otherwise pass the container id from `list_containers`.

A whole document goes in with `add_document`: the file lands in the Memory's upload folder and
its learning turn starts in the background. The call answers `202` with a `document_id`; poll
`GET /v1/documents/{id}` until `learned` is true.

```python
doc = mb.add_document(container="mv-…", title="Call with Acme",
                      content=transcript, custom_id="call-2026-09-17")
```

## 4. Search it

{% tabs %}
{% tab title="Python" %}
```python
found = mb.search_memories("what did we decide about the ledger", limit=5)
for hit in found["results"]:
    print(hit["container_name"], "·", hit["content"])
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
const found = await mb.searchMemories({ q: "what did we decide about the ledger", limit: 5 });
for (const hit of found.results) console.log(hit.container_name, "·", hit.content);
```
{% endtab %}

{% tab title="curl" %}
```bash
curl -s "$BASE/v1/search" -H "Authorization: Bearer $MEMBASE_API_KEY" \
  -H 'content-type: application/json' \
  -d '{"q": "what did we decide about the ledger", "limit": 5}'
```
{% endtab %}
{% endtabs %}

The answer lists passages most relevant first, each naming the container it came from:

```jsonc
{ "query": "…",
  "results": [ { "content": "We settled on Postgres for the ledger.", "score": 1.0,
                 "container": "mv-…", "container_name": "Project decisions", "source": "agent:…" } ],
  "containers": [ { "id": "mv-…", "name": "Project decisions" } ] }
```

A search is a turn inside the user's own memory. The first one after a quiet spell can take up
to a minute while the memory wakes; `containers[]` marks any container that could not answer
yet. Search is retrieval, not an answer; `POST /v1/ask` is the agent's answer.

## The core loop

```
get_profile (once)  →  search_memories (per question)  →  answer, citing container_name
                     →  add_memory / add_document (what the conversation produced)
```

## Three products, one memory

| | What it is | Start here |
|---|---|---|
| **Memory Platform** | The hosted memory layer at app.membase.io. Each account's memory lives in its own agent container; this API, the SDKs and MCP all read and write it. | [Memory Platform](memory-platform.md) |
| **Plugins & MCP** | The same memory inside ChatGPT, Claude, Cursor, Codex, VS Code and any MCP client: one server URL, a consent screen, no code. | [Plugins & MCP](plugins-mcp.md) |
| **Local Memory** | The memory engine on your own machine, for privacy and data control. | [Benchmarks](benchmarks.md) for what the engine does; setup docs come with the local release |

## If something looks wrong

| Answer | Meaning | Do |
|---|---|---|
| `403 … may not use that container` | the key does not reach it, or it does not exist | switch it on under Reach on the key's page, or list containers first |
| `403 … not in this agent's capability profile` | the verb is above the key's access level | mint a key at a higher level; the destructive verbs need Full access |
| `status: confirmation_required` | a destructive verb without `confirm=true` | pass `confirm=true` after the person agreed |
| `422 … no agent containers` / `no model` | the account's memory cannot run a turn on this deployment | the user sets a model under AI Setup |
| `429` | the account's turn budget is spent for now | retry later |
| `document_id: null` on a 202 | the folder sync had not written the row yet | list the container's documents in a moment |
