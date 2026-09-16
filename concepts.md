# Concepts

Six words cover everything on screen. Each is one object in the product, and the left rail
names most of them.

```text
SOURCES                        MEMORY                     APPS
Space folders   ──┐
Uploads         ──┼──▶  Assistant's memory  ──▶  Claude / ChatGPT / Claude Code
Unibase Memory  ──┘         └─ Memories              (Connect)
```

**Assistant.** The one agent that is yours by default. You talk to it on Home, and it holds the
account's memory. Every other object either feeds it or reads from it.

**Memory** (plural *Memories*). A named part of your world, kept up to date: *Project
decisions*, *Reading notes*, *Customers*. A memory has an instruction (what it keeps), sources
(what it reads), a schedule (how often it re-reads) and a list of apps that may use it. It is
private until you let an app use it.

**Source.** Something you handed a memory to learn from. Today that is a folder in your Space,
files you upload, or your conversations with other assistants imported through the Unibase
Memory browser extension. A source has its own page with its own status, and a memory only
learns from it when the memory runs.

**Space.** Your files. A real folder tree the assistant shares with you. Any folder in it can
become a source. The system folders *Generated*, *Assets* and *Trash* sit beside your own.

**Connect / Apps.** The AI apps allowed to read your memories: Claude, ChatGPT, Claude Code,
Codex, Cursor, Windsurf, VS Code. Connecting an app is done once per account. Which memories it
may read is a switch you flip on either side, the app's page or the memory's page.

**Agents.** Agents you build yourself, beyond the assistant. Most people never need this page;
it is where an agent's permissions, tools and endpoint live.

**Marketplace.** Where memories are sold and bought. A listing is live access to one memory
through the buyer's own credential; what they can read is exactly what the seller lets that
memory show. Skills, packaged abilities for agents, are the other half of the market.

## What runs when

Adding a source never runs the memory. A memory learns when you press **Run** (or **Update
now**, the same button while a source holds material the memory has not read), or on its
schedule. A source's sync only fetches raw material; the memory reads it on the next run.

## Two things that are not concepts

*Views*, *groups* and *MCP* appear in API paths and in some older links. On screen a view or
group is simply a **Memory**, and MCP is the protocol behind **Connect**. You do not need
either word to use the product.
