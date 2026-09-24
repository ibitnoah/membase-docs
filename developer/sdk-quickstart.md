---
description: The Python and TypeScript clients. One client, one key, your memory.
---

# SDK Quickstart

The SDK is a thin client over the Membase API. Every method is one operation of the
[API Reference](api-reference.md), and the access level, reach and confirmation rules are
enforced server-side, so the SDK cannot do anything the key cannot.

## Install

{% tabs %}
{% tab title="Python" %}
```bash
pip install membase-sdk
```

Python 3.10 or newer. One dependency, `httpx`.
{% endtab %}

{% tab title="TypeScript" %}
```bash
npm install @membase/sdk
```

Node 18 or newer (global `fetch`), Deno or Bun. Zero dependencies; responses are typed.
{% endtab %}
{% endtabs %}

## Who owns the memory

In Membase **the person owns the memory**, not the app. A developer key is minted by the
account's owner under **Connect › Developer keys**, and it carries what they chose: an access
level, the Memories it may reach, whether it may read their profile, and when it expires. The
SDK takes that key and nothing else.

If you have used a memory API where you tag content with a *container tag* per end user and
one master key reaches them all: that is not this. A Membase **container** is one of the
person's own Memories (a topic), not a tenant. A product that serves many people gives each of
them their own Membase account, and reaches their memory with their own key or their own
consent. See [Memory Platform](memory-platform.md#one-account-per-person).

## Create a client

{% tabs %}
{% tab title="Python" %}
```python
from membase import Membase

client = Membase()                              # MEMBASE_API_KEY from the environment
client = Membase(api_key="mbk_…")               # or explicit
client = Membase(base_url="http://localhost:8080")   # a self-hosted Membase
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
import { Membase } from "@membase/sdk";

const client = new Membase();                                   // MEMBASE_API_KEY from the environment
const client = new Membase({ apiKey: "mbk_…" });
const client = new Membase({ baseUrl: "http://localhost:8080" }); // a self-hosted Membase
```
{% endtab %}
{% endtabs %}

Options: `timeout` (90 s by default, because the first search after a quiet spell waits for
the memory to wake), `max_retries` (2; on 429 and 5xx, with backoff and `Retry-After`).

## The four verbs

{% tabs %}
{% tab title="Python" %}
```python
# add: a document (its text, or a public url) into a Memory. Returns at once, learned in the background.
doc = client.add("Call notes with Acme: they want SSO before the pilot.",
                 container="mv-…", title="Call with Acme", custom_id="call-2026-09-24")
doc["status"]                                   # "queued"; the same custom_id again is a no-op

# search: passages across every Memory in reach, most relevant first, each naming its container
hits = client.search("what does Acme need before the pilot", limit=5)
for h in hits["results"]:
    print(h["container_name"], "·", h["content"])

# profile: who the user is (needs the profile tick on the key)
p = client.profile(q="working hours")
p["static"], p["dynamic"], p["results"]

# ask: the exposed agent's answer (an agent-endpoint credential, e.g. a marketplace subscription)
client.ask("Summarise what the seller learned about Postgres RLS.")
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
const doc = await client.add({ container: "mv-…", content: "Call notes with Acme: they want SSO before the pilot.",
                               title: "Call with Acme", customId: "call-2026-09-24" });
doc.status;                                      // "queued"

const hits = await client.search({ q: "what does Acme need before the pilot", limit: 5 });
for (const h of hits.results) console.log(h.container_name, "·", h.content);

const p = await client.profile({ q: "working hours" });
p.static; p.dynamic; p.results;

await client.ask({ message: "Summarise what the seller learned about Postgres RLS." });
```
{% endtab %}
{% endtabs %}

`add` hands raw material to the Memory's folder and starts its learning turn; poll
`documents.get(id)` until `learned` is true if you need to know. `search` is retrieval, not an
answer; `ask` is the answer.

## The resources

{% tabs %}
{% tab title="Python" %}
```python
client.containers.list()                        # {"containers": [{id, name, description, …}]}

client.documents.list(container="mv-…")        # newest first, each with "learned"
client.documents.get("srcitem_…")
client.documents.delete("srcitem_…", confirm=True)

client.memories.add("We settled on Postgres.", container="mv-…")   # a note the Memory reads
client.memories.add("The user prefers dark mode.", static=True)     # a standing fact → the profile
client.memories.forget("12", container="mv-…", confirm=True)

client.rules()                                  # the user's standing rules for this credential
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
await client.containers.list();

await client.documents.list({ container: "mv-…" });
await client.documents.get("srcitem_…");
await client.documents.delete("srcitem_…", { confirm: true });

await client.memories.add({ content: "We settled on Postgres.", container: "mv-…" });
await client.memories.add({ content: "The user prefers dark mode.", static: true });
await client.memories.forget("12", { container: "mv-…", confirm: true });

await client.rules();
```
{% endtab %}
{% endtabs %}

`delete` and `forget` need a key at **Full access**. Without `confirm` they do not fail: the
answer is `status: confirmation_required` with a `how` sentence to relay to the person. Pass
`confirm` only after they agreed; from a developer key that counts as the owner's confirmation.

## Errors

One class per HTTP status, all carrying the API's error envelope: `code` (which rule refused),
`message`, `details`, `retryable` and `trace_id` (quote it when reporting a problem).

| Status | Class | When |
|---|---|---|
| 400 | `BadRequestError` | a malformed request: a missing `q`, both `content` and `url`, an ambiguous `container` |
| 401 | `AuthenticationError` | no bearer at all |
| 403 | `PermissionDeniedError` | `code: unauthorized`: an unknown, expired or revoked key; a container outside the key's reach; a verb above its level |
| 404 | `NotFoundError` | an unknown document or memory id |
| 422 | `UnprocessableEntityError` | `code: capability_unavailable`: the account's memory cannot run a turn here (no agent container, no model) |
| 429 | `RateLimitError` | the account's turn budget; retried automatically, then raised |
| 5xx | `InternalServerError` | retried automatically, then raised |
| — | `APIConnectionError`, `APITimeoutError` | the request never got an answer |

{% tabs %}
{% tab title="Python" %}
```python
from membase import PermissionDeniedError, RateLimitError

try:
    client.search("x", container="mv-other")
except PermissionDeniedError as e:
    print(e.code, e.message, e.trace_id)       # unauthorized  may not use that container  …
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
import { PermissionDeniedError } from "@membase/sdk";

try {
  await client.search({ q: "x", container: "mv-other" });
} catch (e) {
  if (e instanceof PermissionDeniedError) console.log(e.code, e.message, e.traceId);
}
```
{% endtab %}
{% endtabs %}

## Rules of the road

* **Reach is account state.** The Memories a key may use are switched on the Connect page. A key narrowed there answers `403` on its very next call, with the same token.
* **Confirm means the person agreed.** Never pass `confirm` on your own initiative.
* **Writes are raw material.** `add` hands bytes to the Memory's folder; its agent reads them in the next learning turn.
* **Do not store to filter later.** There are no server-side metadata filters on search; put what matters in the content, and use `custom_id` and `metadata` for your own bookkeeping.
* **The token stays out of logs.** Log the key's hint (`mbk_7f3a92d1…`) from the key's page, never the token.

## Method map

| SDK | REST | Protocol tool | Level |
|---|---|---|---|
| `add(...)` | `POST /v1/documents` | `add_document` | Read & write |
| `search(q, ...)` | `POST /v1/search` | `search_memories` | Read |
| `profile(q)` | `GET /v1/profile` | `get_profile` | Read + profile tick |
| `ask(message)` | `POST /v1/ask` | `ask_agent` | agent exposure |
| `rules()` | `GET /v1/rules` | `memory_rules` | Read |
| `containers.list()` | `GET /v1/containers` | `list_containers` | Read |
| `documents.list(container)` | `GET /v1/documents` | `list_documents` | Read |
| `documents.get(id)` | `GET /v1/documents/{id}` | `get_document` | Read |
| `documents.delete(id, confirm)` | `DELETE /v1/documents/{id}` | `delete_document` | Full access |
| `memories.add(content, ...)` | `POST /v1/memories` | `add_memory` | Read & write |
| `memories.forget(id, ...)` | `DELETE /v1/memories/{id}` | `forget_memory` | Full access |

The MCP server offers the same operations under the protocol tool names, so a model that
learned `search_memories` in Claude is calling what your code calls `search`.

## Without the SDK

Any HTTP client works, and the OpenAPI document generates a typed client in any language:

```bash
npx openapi-typescript https://www.app.membase.io/plugin/openapi/agent-protocol.json -o membase.d.ts
```
