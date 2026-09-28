---
description: Put a person's memory behind a model. The same three calls from the Claude API, the OpenAI API, any tool-calling framework, and any MCP-capable agent.
---

# API Integrations

Every integration is the same loop with a different harness around it:

```
get_profile (once)  →  search_memories (per question)  →  answer, citing container_name
                     →  add_memory / add_document (what the conversation produced)
```

The memory side is always the Membase SDK or REST with the user's developer key
([Authentication & Scopes](authentication.md)). The model side is whatever you already use.
Pick the shape that matches it.

| Your harness | Shape | Writes? |
|---|---|---|
| Claude API | point the request at Membase's MCP server with the key as its token; no tool code | with a Read & write key |
| Claude API, own tools | wrap the SDK in `@beta_tool` / `betaZodTool` and let the tool runner loop | yes |
| OpenAI API, or any function-calling model | declare `search_memories` and `add_memory` as functions, call the SDK when the model asks | yes |
| an agent framework that speaks MCP | give it the server URL and the key as a header; it discovers the tools | with a Read & write key |
| an agent that reads pages and runs commands | the skill: one sentence, no code of yours | up to the key's level |

Everything below assumes `MEMBASE_API_KEY` in the environment, minted at **Read & write**
with **Your profile** ticked, and your model provider's key beside it.

## Claude API: Membase as a remote MCP server

The Claude API can call a remote MCP server on your behalf. Name Membase's server, hand it the
developer key as the authorization token, and Claude gets the key's tools without a line of
tool code:

{% tabs %}
{% tab title="Python" %}
```python
import os
import anthropic

client = anthropic.Anthropic()

response = client.beta.messages.create(
    model="claude-opus-5",
    max_tokens=16000,
    betas=["mcp-client-2025-11-20"],
    mcp_servers=[{
        "type": "url",
        "url": "https://api.app.membase.io/mcp-http",
        "name": "membase",
        "authorization_token": os.environ["MEMBASE_API_KEY"],
    }],
    tools=[{"type": "mcp_toolset", "mcp_server_name": "membase"}],
    system="Read get_profile first. Use search_memories before answering about the user's "
           "past work or decisions, and cite the container_name of anything you use.",
    messages=[{"role": "user", "content": "What did we decide about the ledger, and why?"}],
)

for block in response.content:
    if block.type == "text":
        print(block.text)
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const response = await client.beta.messages.create({
  model: "claude-opus-5",
  max_tokens: 16000,
  betas: ["mcp-client-2025-11-20"],
  mcp_servers: [{
    type: "url",
    url: "https://api.app.membase.io/mcp-http",
    name: "membase",
    authorization_token: process.env.MEMBASE_API_KEY!,
  }],
  tools: [{ type: "mcp_toolset", mcp_server_name: "membase" }],
  system: "Read get_profile first. Use search_memories before answering about the user's " +
          "past work or decisions, and cite the container_name of anything you use.",
  messages: [{ role: "user", content: "What did we decide about the ledger, and why?" }],
});

for (const block of response.content) if (block.type === "text") console.log(block.text);
```
{% endtab %}
{% endtabs %}

The tool list Claude sees is the key's: a Read key offers `list_containers`,
`search_memories`, `get_profile`, `list_documents`, `memory_rules`; a Read & write key adds
`add_memory` and `add_document`. To keep a conversation read-only, mint a Read key for it.

## Claude API: your own tools over the SDK

When you want to shape what the model sees (a smaller tool surface, your own descriptions,
a result format of your choosing), wrap the SDK and let the tool runner drive the loop:

{% tabs %}
{% tab title="Python" %}
```python
import anthropic
from anthropic import beta_tool
from membase import Membase

memory = Membase()                       # MEMBASE_API_KEY
claude = anthropic.Anthropic()

@beta_tool
def search_memories(q: str) -> str:
    """Search the user's memory. Use it before answering anything about their past work,
    decisions or preferences.

    Args:
        q: the question, in natural language.
    """
    hits = memory.search(q, limit=5)["results"]
    return "\n".join(f"[{h['container_name']}] {h['content']}" for h in hits) or "nothing found"

@beta_tool
def remember(content: str) -> str:
    """Save one fact the user asked to remember, in their own words.

    Args:
        content: the fact.
    """
    memory.memories.add(content)
    return "saved"

profile = memory.profile()
runner = claude.beta.messages.tool_runner(
    model="claude-opus-5",
    max_tokens=16000,
    system="Standing facts about the user: " + "; ".join(profile["static"]),
    tools=[search_memories, remember],
    messages=[{"role": "user", "content": "Remind me why we picked Postgres, then note that we'll revisit it in Q1."}],
)
final = runner.until_done()
print(next(b.text for b in final.content if b.type == "text"))
```
{% endtab %}

{% tab title="TypeScript" %}
```ts
import Anthropic from "@anthropic-ai/sdk";
import { betaZodTool } from "@anthropic-ai/sdk/helpers/beta/zod";
import { z } from "zod";
import { Membase } from "@membase/sdk";

const memory = new Membase();            // MEMBASE_API_KEY
const claude = new Anthropic();

const searchMemories = betaZodTool({
  name: "search_memories",
  description: "Search the user's memory. Use it before answering anything about their past work, decisions or preferences.",
  inputSchema: z.object({ q: z.string().describe("the question, in natural language") }),
  run: async ({ q }) => {
    const { results } = await memory.search({ q, limit: 5 });
    return results.map((h) => `[${h.container_name}] ${h.content}`).join("\n") || "nothing found";
  },
});

const remember = betaZodTool({
  name: "remember",
  description: "Save one fact the user asked to remember, in their own words.",
  inputSchema: z.object({ content: z.string() }),
  run: async ({ content }) => { await memory.memories.add({ content }); return "saved"; },
});

const profile = await memory.profile();
const final = await claude.beta.messages.toolRunner({
  model: "claude-opus-5",
  max_tokens: 16000,
  system: "Standing facts about the user: " + profile.static.join("; "),
  tools: [searchMemories, remember],
  messages: [{ role: "user", content: "Remind me why we picked Postgres, then note that we'll revisit it in Q1." }],
});
for (const block of final.content) if (block.type === "text") console.log(block.text);
```
{% endtab %}
{% endtabs %}

`remember` calls `memories.add` without a container, which works while exactly one Memory is
in the key's reach; pass `container=` once there are several.

## OpenAI API, or any function-calling model

Declare the two verbs as functions and call the SDK when the model asks. The shape is the
Chat Completions tool-calling loop; any provider with the same loop takes the same two
definitions.

```ts
import OpenAI from "openai";
import { Membase } from "@membase/sdk";

const memory = new Membase();
const openai = new OpenAI();

const tools: OpenAI.Chat.Completions.ChatCompletionTool[] = [
  { type: "function", function: {
      name: "search_memories",
      description: "Search the user's memory. Use it before answering anything about their past work, decisions or preferences.",
      parameters: { type: "object", properties: { q: { type: "string" } }, required: ["q"] } } },
  { type: "function", function: {
      name: "add_memory",
      description: "Save one fact the user asked to remember, in their own words.",
      parameters: { type: "object", properties: { content: { type: "string" } }, required: ["content"] } } },
];

async function runTool(name: string, args: Record<string, string>): Promise<string> {
  if (name === "search_memories") {
    const { results } = await memory.search({ q: args.q, limit: 5 });
    return JSON.stringify(results.map((h) => ({ memory: h.container_name, content: h.content })));
  }
  await memory.memories.add({ content: args.content });
  return "saved";
}

const profile = await memory.profile();
const messages: OpenAI.Chat.Completions.ChatCompletionMessageParam[] = [
  { role: "system", content: "Standing facts about the user: " + profile.static.join("; ") },
  { role: "user", content: "What did we decide about the ledger?" },
];

for (;;) {
  const turn = await openai.chat.completions.create({ model: process.env.OPENAI_MODEL!, messages, tools });
  const reply = turn.choices[0].message;
  messages.push(reply);
  if (!reply.tool_calls?.length) { console.log(reply.content); break; }
  for (const call of reply.tool_calls) {
    const content = await runTool(call.function.name, JSON.parse(call.function.arguments));
    messages.push({ role: "tool", tool_call_id: call.id, content });
  }
}
```

The same loop in Python is the SDK's `client.search(...)` and `client.memories.add(...)`
behind two function definitions; nothing else changes.

## An agent framework that speaks MCP

A framework with an MCP client (LangChain's MCP adapters, the OpenAI Agents SDK, Pydantic AI,
Mastra, and most others) needs no Membase-specific code. Give it the server as a Streamable
HTTP endpoint with the key in the header:

```json
{ "url": "https://api.app.membase.io/mcp-http",
  "headers": { "Authorization": "Bearer mbk_…" } }
```

It discovers the key's tools with `tools/list` and calls them by the names in the
[API Reference](api-reference.md). Where the framework's own MCP client speaks OAuth, leave
the header out and the user consents in the browser instead; the connection is then
read-only, which is what a user-facing agent usually wants.

## An agent that reads pages and runs commands

Claude Code, Codex, a Cursor agent, or your own agent with a shell: no integration code.
Tell it

> Install Membase from https://www.app.membase.io/skill using key mbk_…

and it installs the skill folder, keeps the key as `MEMBASE_API_KEY` and calls the REST API
with it, at the key's level. [Plugins Overview](https://noah-gao.gitbook.io/membase-user-guide/plugins-mcp#the-skill-in-one-sentence).

## Without an SDK

Any HTTP client works, and the OpenAPI document generates a typed client for any language:

```bash
curl -s https://www.app.membase.io/plugin/openapi/agent-protocol.json -o agent-protocol.json
npx openapi-typescript agent-protocol.json -o membase.d.ts
```

Three requests cover most integrations: `GET /v1/profile`, `POST /v1/search` with `{"q": …}`
and `POST /v1/memories` with `{"content": …}`, each with the bearer header.

## Rules that hold in every shape

* **Profile once, search per question.** The profile is small and standing; search is a turn inside the user's container.
* **Cite the container.** Every hit names `container_name`; say where an answer came from.
* **Save what the user supplied,** in their words, and only when they asked or plainly meant to.
* **Never confirm on your own.** A delete or forget without `confirm=true` answers with a `how` sentence; relay it and stop.
* **Treat `403` as withdrawn access.** The owner narrowed or revoked the key; do not retry with it.
* **Expect the first search to be slow.** Up to a minute after a quiet spell; keep the SDK's timeout.
