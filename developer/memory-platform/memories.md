---
description: A Memory is a container. What its page shows, what a run does, and which API calls read and change the same thing.
---

# Memories

A **Memory** is one named part of the person's world, kept up to date: *Project decisions*,
*Reading notes*, *Customers*. In the API it is a **container**. It has an instruction (what
it keeps), sources (what it reads), a schedule (how often it re-reads) and a list of apps and
keys that may use it. It is private until the owner lets a credential use it.

The Memory page is one drive: the root shows Memories as tiles, and each opens as a page
with the same anatomy. The assistant's own memory is always the first tile.

## What a tile says

| Tile word | Meaning | In the API |
|---|---|---|
| *Empty* | no instruction yet | listed, searches answer nothing |
| *Add a source* | reads nothing yet | listed, `list_documents` is empty |
| *On sale* | listed on the Marketplace | buyers hold agent-endpoint credentials to it |
| *Subscribed* | bought from someone else | `ask_agent` only; no documents, no writes |

`list_containers` returns each Memory in the credential's reach as `{id, name,
description, …}`; the `id` is the `container` every other call takes.

## A Memory's page

* **Name**, renamed in place. It is `name` in `list_containers` and `container_name` on every search hit; the id never changes.
* **Run / Update now.** The one way to make the Memory read its sources by hand. It says *Update now* while a source holds unread material, and is disabled until there is a source. `add_document` presses it for you.
* **Settings.** The whole configuration as one dialog; every field saves as it changes.
* **Delete…** Says the irreversible part first: everything the Memory learned goes with it, then what depends on it, including every subscriber's access. There is no API verb for this.
* **Add to this memory.** Three doors, Unibase Memory, Upload Files, Your Files; sources already added are listed under them with a **Remove** per row. [Sources & Files](sources-and-files.md).
* **Your Memory.** What the Memory currently knows: search, a kind filter, one list of pages and facts. Nothing here was typed by hand; it is what the last run produced. Each item is a **memory** in the API, and `forget_memory` removes one.
* **Use in.** One switch per app and per key. It is the same switch as the one on the app's or key's page: reach, seen from the Memory's side.

## The status line

| It says | Meaning | For code |
|---|---|---|
| *Nothing yet* | never run | searches answer nothing |
| *Checking…* | the page is reading the state | |
| *Running…* | a run is queued or running; survives a reload | documents' `learned` is still false |
| *Updated 3h ago · Report* | the last run read the current sources | `learned` is true on everything read |
| *Run failed* | the last run did not finish | the report says why; usually the model source or a source's credential |

"Unread" is decided by versions, not clocks. A sync that changed nothing, or a run on another
Memory, cannot make this one look up to date.

## Settings

* **Instruction.** What the Memory keeps. One field. For a Memory over conversations it is a filter on what the chats say, not a subject to write about.
* **Schedule.** *Manual* until one is set; hourly, daily, weekly, monthly or a custom cron, in UTC. [Schedules](schedules.md).
* **Sharing.** Apps and keys as switches, what depends on this Memory, **Sell** / **Stop selling**.
* **Advanced.** The model this Memory's agent runs on, its skills, its agent, and **Open in Studio**, the pipeline canvas behind the run.

## Reading a Memory from code

```python
containers = client.containers.list()["containers"]           # what the key reaches
decisions = next(c for c in containers if c["name"] == "Project decisions")

hits = client.search("what did we decide about the ledger", container=decisions["id"], limit=5)
for h in hits["results"]:
    print(h["container_name"], "·", h["content"])
```

Omit `container` to search every Memory in reach at once; each hit still names its container.
A search is a turn inside the account's container, so the first one after a quiet spell can
take up to a minute; `containers[]` in the answer marks any that could not answer yet.

## Writing to a Memory from code

```python
client.memories.add("We settled on Postgres for the ledger.", container=decisions["id"])   # a note it reads
client.add(transcript, container=decisions["id"], title="Call with Acme", custom_id="call-2026-09-24")  # a document
```

`memories.add` is a short note; `add` is a document that lands in the Memory's upload folder
and starts a learning turn. Both are raw material until the run commits them. A standing
fact about the person (`static=True`) goes to the profile instead, not to any Memory.

## Forgetting

```python
client.memories.forget("12", container=decisions["id"], confirm=True)
```

Needs a Full access key and, without `confirm`, answers `status: confirmation_required` with
a `how` sentence. Deleting the whole Memory is the owner's, in the app, and has no API verb.

## If something looks wrong

| The person sees | Meaning | Do |
|---|---|---|
| **Run** disabled | no source yet, or state still loading | add a source; wait for *Checking…* |
| *Update now* | unread material waits | press it, or let the schedule |
| *Run failed* | the last run did not finish | open **Report**; usually the model source is off or a source needs reauthorization |
| *Empty* on the tile | no instruction | Settings › Instruction |
| a search from code answers nothing | never run, or the key does not reach it | run it; check **Use in** |
