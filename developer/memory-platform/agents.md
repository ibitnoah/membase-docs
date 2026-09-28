---
description: The agents an account builds beyond its assistant, and the one API verb that reaches an agent, ask_agent, on a credential that exposes it.
---

# Agents

Every account has one agent by default, the **assistant** the person talks to on Home. It
holds the account's memory of record and the profile, and every other object either feeds
it or reads from it. The **Agents** page is for the agents the person builds beyond it. Most
accounts never need it.

## What the page shows

One row per agent: its endpoint, its permissions, its tools, and **Revoke**. AI apps that
read the account's Memories are not agents; they live on Connect. A Memory's own agent, the
one that runs its learning turns, is reached from the Memory's Settings under **Advanced**,
not from here.

## Where agents come from

* **A Memory's agent** is made with the Memory and runs its learning turns under its instruction. Its model, skills and pipeline canvas (**Open in Studio**) are the Memory's Advanced settings. Deleting the Memory deletes it.
* **An agent the person builds** on this page, with its own permissions and tools, for something the assistant should not do itself.
* **An agent endpoint** is a Memory's agent exposed to others: sold on the Marketplace, or shared. Each subscriber holds a credential of their own to it.

## Reaching an agent from code

The one protocol verb that talks to an agent is `ask_agent`:

```python
answer = client.ask("Summarise what the seller learned about Postgres RLS.")
```

It answers only on a credential that **exposes** an agent: a marketplace subscription, or a
share the owner made. An ordinary developer key holds no agent, and the call is refused with
`403 · unauthorized · tool 'agent_invoke' is not in this agent's capability profile`. The
reply on a bound credential is the agent's answer from what its Memory learned; the buyer
gets answers, never files.

`ask` is the other kind of read next to `search`: search is retrieval (passages, most
relevant first, each naming its container), ask is an answer. A product that wants to put a
model over the person's memory uses search and its own model
([API Integrations](https://noah-gao.gitbook.io/membase-user-guide/api-integrations)); a product that wants a Memory's own agent to
answer subscribes to it.

`workflow_invoke` belongs to workflow exposures, a canvas run as an endpoint, and is offered
only over MCP on a credential that exposes one.

## Permissions

An agent's permissions are the account's, narrowed: what it may read, what it may write,
which tools it holds. A trusted agent run commits its non-destructive output to the
account's memory automatically, provenance-tagged and reversible. Destructive output (delete,
forget, permission or credential changes) always waits for explicit confirmation and is
never auto-committed. Untrusted sandbox output is never auto-committed.

**Revoke** on the row ends the agent's credentials at once; every holder fails on its next
call.
