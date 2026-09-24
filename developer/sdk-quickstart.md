---
description: The Python and TypeScript clients. One method per API operation, the same names, the same rules.
---

# SDK Quickstart

The SDK is a thin client over the REST API. Every method is one operation of the
[API Reference](api-reference.md) under the same name, and the access level, reach and
confirmation rules are enforced server-side, so the SDK cannot do anything the key cannot.

## Install

{% tabs %}
{% tab title="Python" %}
```bash
pip install membase-sdk
```

Python 3.10 or newer. The only dependency is an HTTP client.
{% endtab %}

{% tab title="TypeScript" %}
```bash
npm install @membase/sdk
```

Node 18 or newer, or any runtime with `fetch`. Types are generated from the OpenAPI document,
so responses are typed.
{% endtab %}
{% endtabs %}

## Create a client

{% tabs %}
{% tab title="Python" %}
```python
from membase_sdk import Membase

mb = Membase()                                  # MEMBASE_API_KEY from the environment
mb = Membase(api_key="mbk_…")                   # or explicit
mb = Membase(api_key="mbk_…", base_url="https://api.app.membase.io")   # the default base
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
import { Membase } from "@membase/sdk";

const mb = new Membase();                                   // MEMBASE_API_KEY from the environment
const mb = new Membase({ apiKey: "mbk_…" });
const mb = new Membase({ apiKey: "mbk_…", baseUrl: "https://api.app.membase.io" });
```
{% endtab %}
{% endtabs %}

The key comes from **Connect › Developer keys** in the app ([API Quickstart, step 1](api-quickstart.md#1-get-an-api-key)).

## Read

{% tabs %}
{% tab title="Python" %}
```python
mb.list_containers()                            # {"containers": [{id, name, description, …}]}
mb.search_memories("pricing decision in August", container=None, limit=5)
mb.get_profile(q="working hours")               # {"static": [...], "dynamic": [...], "results": [...]}
mb.list_documents(container="mv-…")
mb.get_document("srcitem_…")                    # {…, "learned": true}
mb.memory_rules()                               # the user's standing rules for this credential
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
await mb.listContainers();
await mb.searchMemories({ q: "pricing decision in August", limit: 5 });
await mb.getProfile({ q: "working hours" });
await mb.listDocuments({ container: "mv-…" });
await mb.getDocument("srcitem_…");
await mb.memoryRules();
```
{% endtab %}
{% endtabs %}

`get_profile` answers only when the key was minted with the profile tick. `search_memories`
without `container` searches every container in reach and names the container on each hit.

## Write

Needs a key at **Read & write** or above.

{% tabs %}
{% tab title="Python" %}
```python
mb.add_memory("The user prefers dark mode.", static=True)     # a standing fact → the profile
mb.add_memory("We settled on Postgres.", container="mv-…")    # a note the container reads

doc = mb.add_document(container="mv-…", title="Call with Acme",
                      content=transcript, custom_id="call-2026-09-17")
# → {"document_id": "srcitem_…", "status": "queued", "learning_run": {...}}
doc = mb.add_document(container="mv-…", url="https://example.com/spec.pdf")
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
await mb.addMemory({ content: "The user prefers dark mode.", static: true });
await mb.addMemory({ content: "We settled on Postgres.", container: "mv-…" });

const doc = await mb.addDocument({ container: "mv-…", title: "Call with Acme",
                                   content: transcript, customId: "call-2026-09-17" });
```
{% endtab %}
{% endtabs %}

`add_document` hands raw material to the container's folder and starts its learning turn; the
call returns at once with `status: queued`. Poll `get_document` for `learned`. The same
`custom_id` sent again is a no-op, so retries are safe.

## Remove

Needs a key at **Full access**, and every call must carry `confirm=True`. Without it the call
does not fail; it answers `status: confirmation_required` with a `how` sentence to relay to
the person. Pass `confirm` only after they agreed.

{% tabs %}
{% tab title="Python" %}
```python
mb.delete_document("srcitem_…", confirm=True)          # the file goes to the Files trash
mb.forget_memory("12", container="mv-…", confirm=True)
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
await mb.deleteDocument("srcitem_…", { confirm: true });
await mb.forgetMemory("12", { container: "mv-…", confirm: true });
```
{% endtab %}
{% endtabs %}

## Ask the agent

`ask_agent` belongs to an agent exposure (a Memory sold or shared as an agent endpoint), not to
a developer key. With such a credential:

```python
mb.ask_agent("What did the seller learn about Postgres RLS?")
```

## Naming

| Operation | Python | TypeScript | REST |
|---|---|---|---|
| `list_containers` | `list_containers()` | `listContainers()` | `GET /v1/containers` |
| `search_memories` | `search_memories(q, container=, limit=)` | `searchMemories({q, container, limit})` | `POST /v1/search` |
| `get_profile` | `get_profile(q=)` | `getProfile({q})` | `GET /v1/profile?q=` |
| `list_documents` | `list_documents(container=)` | `listDocuments({container})` | `GET /v1/documents?container=` |
| `get_document` | `get_document(id)` | `getDocument(id)` | `GET /v1/documents/{id}` |
| `memory_rules` | `memory_rules()` | `memoryRules()` | `GET /v1/rules` |
| `add_memory` | `add_memory(content, container=, static=, title=)` | `addMemory({content, container, static, title})` | `POST /v1/memories` |
| `add_document` | `add_document(container=, content=, url=, title=, metadata=, custom_id=)` | `addDocument({container, content, url, title, metadata, customId})` | `POST /v1/documents` |
| `delete_document` | `delete_document(id, confirm=)` | `deleteDocument(id, {confirm})` | `DELETE /v1/documents/{id}?confirm=` |
| `forget_memory` | `forget_memory(id, container=, confirm=)` | `forgetMemory(id, {container, confirm})` | `DELETE /v1/memories/{id}?container=&confirm=` |
| `ask_agent` | `ask_agent(message, model=)` | `askAgent({message, model})` | `POST /v1/ask` |

## Errors

The API answers one status per class, in the platform's error envelope
(`{"error": {"code", "message", "details", "retryable", "trace_id"}}`). The SDK raises one
exception type per row.

| Status | Code | When |
|---|---|---|
| 400 | `validation` | a missing `q`, both `content` and `url`, an ambiguous `container` |
| 403 | `unauthorized` | the container is outside the credential's reach, or the verb above its level |
| 404 | `not_found` | an unknown document or memory id |
| 422 | `capability_unavailable` | the account's memory cannot run a turn on this deployment (no agent container, no model) |
| 429 | `rate_limited` | the account's concurrent-turn budget; retry later |

Include `trace_id` when reporting a problem.

## Rules of the road

* **Reach is account state.** The Memories a key may use are switched on the Connect page. A key narrowed there answers `403` on its very next call, with the same token.
* **Confirm means the person agreed.** From a developer key, `confirm=true` counts as the owner's confirmation, because the holder is the account acting through code it wrote.
* **Writes are raw material.** `add_document` hands bytes to the container's folder; the container's agent reads them in its next learning turn. Poll `learned` if you need to know.
* **Do not store to filter later.** There are no server-side metadata filters on search; put what matters in the content, and use `custom_id` and `metadata` for your own bookkeeping.
* **One account per person.** A container is a topic, not an end user. A product serving many end users gives each their own Membase account and their own key or consent.

## Without the SDK

Any HTTP client works, and the OpenAPI document generates a typed client in any language:

```bash
npx openapi-typescript https://www.app.membase.io/plugin/openapi/agent-protocol.json -o membase.d.ts
```
