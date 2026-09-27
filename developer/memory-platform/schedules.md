---
description: Every scheduled task in one place. A Memory's cadence, a scheduled agent, what pausing means, and what code sees when a schedule ran.
---

# Schedules

A **schedule** makes a Memory read its sources on a cadence instead of waiting for the person
to press **Run**. Every scheduled task in the account is on this page: each Memory's cadence,
set in its Settings, and any agent the person put on a schedule.

## Setting one

In the Memory's Settings, **Schedule** is *Manual* until one is set. Cadences are picked, not
typed: hourly, daily, weekly, monthly, or a custom cron. Times are UTC; the picker shows the
local equivalent beside it. The field saves as it changes.

## Pause, resume, retune

The Schedules page pauses, resumes or retunes a live cadence without opening the Memory. A
schedule has no delete verb: pausing is how it stops, and the row says *paused* rather than
pretending it is gone. Setting the Memory back to *Manual* removes the cadence from the page.

## What a scheduled run does

Exactly what **Run** does: the Memory's agent reads the material its sources hold that it
has not read yet, under the instruction, and commits what it learned. A run on a schedule
with nothing unread does nothing and does not make the Memory look newer; "unread" is
decided by versions, not by the clock.

A scheduled run's result stays on its run record, on the Activity page and on this page, and
is sent to the Telegram chat when one is bound to the assistant.

## What code sees

Nothing changes in the API when a schedule fires, except the memory itself: documents'
`learned` turns true, and the next `search_memories` answers from what the run learned. A
product that adds documents with `add_document` does not need a schedule for them; the call
starts a run on its own. A schedule matters for material that arrives without a call, such as
a folder the person keeps adding to, or a Notion selection that changes.

There is no API to create or pause a schedule; it is the owner's, in the app.

## If something looks wrong

| It says | What it means | Do |
|---|---|---|
| *paused* | the schedule exists and does not fire | **Resume** |
| the time looks wrong | schedules run in UTC; the picker shows local time beside it | nothing, or retune |
| the run fired but the Memory says *Run failed* | the run did not finish | the Memory's **Report**; usually the model source or a source's credential |
| a scheduled result never reached Telegram | no chat is bound to the assistant | Home › Remote › **Connect Telegram** |
