---
description: "What ran, in one timeline: Memory runs, syncs, scheduled tasks and agent turns. And where a key's own activity lives."
---

# Activity

The Activity page is a human-readable timeline of what ran on the account: Memory runs,
source syncs, scheduled tasks, agent turns. Each entry says what it was, when, and how it
ended; a run's entry opens its **Report**, the same one the Memory's status line links to.

## What is on it

| Entry | Comes from | Ends with |
|---|---|---|
| a Memory run | **Run**, a schedule, or `add_document` | what it learned, or why it failed |
| a source sync | **Sync now**, or a connector's own cadence | raw material fetched, or a sync error |
| a scheduled task | [Schedules](schedules.md) | its result; also sent to Telegram when a chat is bound |
| an agent turn | the assistant, or an agent the person built | the turn's outcome |

A scheduled run's result stays on its run record here and on Schedules.

## Where a key's activity lives

Calls made with a developer key are not on this page; they are on the **key's page** under
Connect › Developer keys, which keeps them per key:

* **Usage.** Calls in the last 30 or 7 days, how many were *refused* (the key asked for a Memory or a verb it does not have), and the tool it calls most.
* **Recent activity.** The last calls one by one: tool, Memory, what came back, when. A refused call is red and says why.

The same numbers are on `GET /v1/api-keys/{id}/usage`, with the owner's session. An app
connected over MCP shows its last activity on its own page under Connect.

## What a `202` from `add_document` becomes

The call answers at once with `status: queued` and a `document_id`. The run it started is a
Memory run on this page; when the entry ends, the document's `learned` is true and the next
search answers from it. If the entry ends in *Run failed*, the document is still listed and
still unread, and the report says why; usually the account has no working model under
[AI Setup](ai-setup.md) or a source needs reauthorization.
