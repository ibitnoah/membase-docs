---
title: Claude
description: "Membase in Claude on the web and the desktop app, as a custom connector with OAuth consent in the browser."
---

# Claude

Claude (claude.ai and the desktop app) takes Membase as a **custom connector** and reads the
Memories you switch on for it. For Claude Code, the command-line agent, see
[Claude Code](claude-code.md).

## Before you start

* A Claude plan that allows custom connectors (organisation admins can restrict them).
* A Membase account with at least one Memory that has run once.

## Set up

1. **Customize → Connectors → + → Add custom connector**.
2. Paste `https://api.app.membase.io/mcp-http` and confirm.
3. Finish the authorization in the browser. On Membase's consent screen tick the Memories Claude may use and, if it may read the profile, **Let it know about you**.
4. In a chat, make sure the connector is enabled under the tools menu, then ask *"List my Membase containers."*

## What it can do

`list_containers`, `search_memories` and, when ticked, `get_profile`. Nothing that writes: a
request to save, delete or forget answers with a sentence for Claude to relay. Claude's
connector settings have no field for a header, so the consent path is the only one here; a
developer key belongs to [Claude Code](claude-code.md).

## Which Memories it uses

An empty container list means no Memory is switched on for this connection. Switch one on
under **Uses** on Claude's page in Connect, or under **Use in** on the Memory. Off takes
effect on Claude's next question.

## Remove it

Remove the connector on the same Connectors page, and **Disconnect…** on Claude's page in
Membase's Connect too: the client does not tell the server.

## If something looks wrong

| It says | What it means | Do |
|---|---|---|
| the connector never finishes authorizing | the URL has a trailing slash or a typo, or the consent tab was closed | paste the URL exactly; add the connector again |
| *List my containers* answers an empty list | connected, no Memory switched on | **Uses** on Claude's page in Connect |
| the last step lands on the Membase app | the authorization completed anyway | go back to the Claude tab |
| the model says a Memory "could not answer yet" | the memory was asleep; the first search wakes it | ask again in a moment |
| the model answers from nothing about you | the profile tick is off | tick **Let it know about you** |
