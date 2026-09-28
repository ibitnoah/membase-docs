---
title: Build with Membase
description: "Start with a developer key and the SDK, then put the same memory behind your own model, an agent framework or an AI coding tool."
---

# Build with Membase

One client, one key, the user's memory. Every call runs against `https://api.app.membase.io`
with a developer key the user minted under **Connect › Developer keys**.

<a href="api-quickstart.md" class="button primary">Quickstart</a> <a href="api-reference.md" class="button secondary">API reference</a> <a href="https://www.app.membase.io" class="button secondary">Dashboard</a> <a href="https://github.com/unibaseio" class="button secondary">GitHub</a> <a href="https://chromewebstore.google.com/detail/edmncknbiihfoakimejbepnaeemaaamf" class="button secondary">Install Extension</a>

{% tabs %}
{% tab title="Python" %}
```python
from membase import Membase          # pip install membase-sdk

client = Membase()                    # MEMBASE_API_KEY, from Connect › Developer keys

client.add("Call notes with Acme: they want SSO before the pilot.",
           container="mv-…", custom_id="call-2026-09-24")

for hit in client.search("what does Acme need before the pilot", limit=3)["results"]:
    print(hit["container_name"], "·", hit["content"])
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
import { Membase } from "membase-sdk";   // npm install membase-sdk

const client = new Membase();              // MEMBASE_API_KEY, from Connect › Developer keys

await client.add({ container: "mv-…", content: "Call notes with Acme: they want SSO before the pilot.",
                   customId: "call-2026-09-24" });

const { results } = await client.search({ q: "what does Acme need before the pilot", limit: 3 });
for (const hit of results) console.log(hit.container_name, "·", hit.content);
```
{% endtab %}

{% tab title="curl" %}
```bash
export MEMBASE_API_KEY="mbk_…"
BASE=https://api.app.membase.io

curl -s "$BASE/v1/documents" -H "Authorization: Bearer $MEMBASE_API_KEY" \
  -H 'content-type: application/json' \
  -d '{"container": "mv-…", "content": "Call notes with Acme: they want SSO before the pilot.", "custom_id": "call-2026-09-24"}'

curl -s "$BASE/v1/search" -H "Authorization: Bearer $MEMBASE_API_KEY" \
  -H 'content-type: application/json' \
  -d '{"q": "what does Acme need before the pilot", "limit": 3}'
```
{% endtab %}
{% endtabs %}

## How the API thinks

Three nouns, plain verbs. A **container** is one named space of memory (a *Memory* in the
app), a **memory** is one fact a container holds, a **document** is one piece of raw material a
container has read. You `list`, `search`, `get`, `add`, `delete` and `forget` them.

A credential is always the user's own: a developer key they minted, or the consent they gave
an app. It reaches the containers they ticked, at the access level they chose, and both can be
changed live on the Connect page without re-minting. Nothing is ever deleted without an
explicit `confirm=true`.

## Where to go

| Page | What it gives you |
|---|---|
| [Quickstart](api-quickstart.md) | get a key, install the SDK, write a memory, search it: five minutes to the first call |
| [SDK Quickstart](sdk-quickstart.md) | the Python and TypeScript clients: `add`, `search`, `profile`, `ask` and the resources under them |
| [Platform overview](platform-overview.md) | the product around the API: what each screen of the app is to your code |
| [Memory operations](memory-operations.md) | every verb end to end: containers, search, the profile, notes, documents, ask, forget |
| [Authentication & Scopes](authentication.md) | the two credentials, access levels, reach, the profile tick, expiry and revocation |
| [Multi-user isolation](multi-user-isolation.md) | the account is the tenant: what that means for a product that serves many people |
| [How Membase works](how-membase-works.md) | where memory lives, how material becomes memory, why a search is a turn |
| [Benchmarks](benchmarks.md) | LoCoMo, LongMemEval and DMR results for the memory engine, with the method behind each number |
| [Claude API](claude-api.md) · [OpenAI API](openai-api.md) · [MCP frameworks](mcp-frameworks.md) · [AI coding tools](ai-coding-tools.md) | the memory behind a model: pick the harness you already use |
| [API reference](api-reference.md) | every operation with its access level, parameters and response shape; the OpenAPI document |

## Pick a recipe

Every integration is the same loop with a different harness around it:

```
get_profile (once)  →  search_memories (per question)  →  answer, citing container_name
                     →  add_memory / add_document (what the conversation produced)
```

| Your harness | Shape | Page |
|---|---|---|
| Claude API | point the request at Membase's MCP server with the key as its token, or wrap the SDK in your own tools | [Claude API](claude-api.md) |
| OpenAI API, or any function-calling model | declare `search_memories` and `add_memory` as functions, call the SDK when the model asks | [OpenAI API](openai-api.md) |
| an agent framework that speaks MCP | give it the server URL and the key as a header; it discovers the tools | [MCP frameworks](mcp-frameworks.md) |
| an AI that reads pages and runs commands | the skill: one sentence, no code of yours; and the same skill teaches it to build on the API | [AI coding tools](ai-coding-tools.md) |

{% hint style="info" %}
Looking for the app itself rather than its API? The **Use Membase** tab covers Home, Memory,
Files, Connect and the rest of the product, one page per screen.
{% endhint %}
