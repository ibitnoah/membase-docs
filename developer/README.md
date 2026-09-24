---
title: Build with Membase
description: Start with the API, then connect the same memory layer through plugins and MCP, or run it locally.
---

# Build with Membase.

Start with the API, then connect the same memory layer through plugins and MCP—or run it locally.

<a href="api-quickstart.md" class="button primary">API Quickstart</a> <a href="api-reference.md" class="button secondary">API Reference</a>

{% tabs %}
{% tab title="Python" %}
```python
from membase_sdk import Membase

mb = Membase(api_key="mbk_…")   # Connect › Developer keys

mb.add_memory("We store money as integer cents, never floats.")

for hit in mb.search_memories("how do we store money", limit=3)["results"]:
    print(hit["container_name"], "·", hit["content"])
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
import { Membase } from "@membase/sdk";

const mb = new Membase({ apiKey: process.env.MEMBASE_API_KEY! });

await mb.addMemory({ content: "We store money as integer cents, never floats." });

const { results } = await mb.searchMemories({ q: "how do we store money", limit: 3 });
for (const hit of results) console.log(hit.container_name, "·", hit.content);
```
{% endtab %}

{% tab title="curl" %}
```bash
export MEMBASE_API_KEY="mbk_…"
BASE=https://api.app.membase.io

curl -s "$BASE/v1/memories" -H "Authorization: Bearer $MEMBASE_API_KEY" \
  -H 'content-type: application/json' \
  -d '{"content": "We store money as integer cents, never floats."}'

curl -s "$BASE/v1/search" -H "Authorization: Bearer $MEMBASE_API_KEY" \
  -H 'content-type: application/json' \
  -d '{"q": "how do we store money", "limit": 3}'
```
{% endtab %}
{% endtabs %}

Every call runs against `https://api.app.membase.io` with a developer key from **Connect ›
Developer keys** in the Membase app. The [API Quickstart](api-quickstart.md) gets you from no
key to a first search in five minutes.

## Three products, one memory

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody>
<tr><td><strong>Memory Platform</strong></td><td>The hosted memory layer. Every account's memory lives in its own agent container; the REST API, the SDKs and MCP read and write the same memory.</td><td><a href="memory-platform.md">memory-platform.md</a></td></tr>
<tr><td><strong>Plugins &#x26; MCP</strong></td><td>Connect that memory to ChatGPT, Claude, Cursor, Codex, VS Code and any MCP client, with one server URL and a consent screen.</td><td><a href="plugins-mcp.md">plugins-mcp.md</a></td></tr>
<tr><td><strong>Local Memory</strong></td><td>Run the memory engine on your own machine for privacy and data control. Documentation comes with the local release.</td><td></td></tr>
</tbody></table>

## Where to go

| Page | What it gives you |
|---|---|
| [API Quickstart](api-quickstart.md) | get a key, install the SDK, write a memory, search it: five minutes to the first call |
| [SDK Quickstart](sdk-quickstart.md) | the Python and TypeScript clients: one method per API operation, same names, same rules |
| [API Reference](api-reference.md) | every operation with its access level, parameters and response shape; the OpenAPI document |
| [Plugins & MCP](plugins-mcp.md) | the MCP server, per-client setup, the skill install sentence, consent and revocation |
| [Memory Platform](memory-platform.md) | how the hosted platform is built: containers, memories, documents; credentials, access and reach; limits |
| [Benchmarks](benchmarks.md) | LoCoMo, LongMemEval and DMR results for the memory engine, with the method behind each number |

## How the API thinks

Three nouns, plain verbs. A **container** is one named space of memory (a *Memory* in the
app), a **memory** is one fact a container holds, a **document** is one piece of raw material a
container has read. You `list`, `search`, `get`, `add`, `delete` and `forget` them.

A credential is always the user's own: a developer key they minted, or the consent they gave
an app. It reaches the containers they ticked, at the access level they chose, and both can be
changed live on the Connect page without re-minting. Nothing is ever deleted without an
explicit `confirm=true`.

{% hint style="info" %}
Looking for the app itself rather than its API? The **User Guide** covers Home, Memory, Files,
Connect and the rest of the product, one page per screen.
{% endhint %}
