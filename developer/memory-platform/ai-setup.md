---
description: Where the account's model comes from. A key of its own or a subscription, verified before use, proxied so no credential enters the container, and what code sees when there is none.
---

# AI Setup

Every learning turn, every search and every assistant answer needs a model. **AI Setup** is
where the account chooses it. Two ways in:

* **Bring your own key** for a provider: OpenAI, Anthropic, Gemini, DeepSeek, Kimi, Qwen, OpenRouter, or any OpenAI-compatible endpoint. Press **Connect**, paste the key, pick a model from the list the key returns. A key with no model is not saved; the catalogue names no models of its own.
* **A Claude or ChatGPT subscription.** Press **Connect** on the subscription card and sign in with that provider. Turns are then billed to the subscription.

The **Connected** tab lists what is set up; one of them is the account's **active** source,
which answers everything that is not an agent turn. Which source the assistant itself runs
on is the assistant's own field, in its settings on Home. A Memory's agent can be given its
own under the Memory's Advanced settings.

## Verify

Each connection is verified with one short real request before it is used. A key that fails
verification is never used, and its row says why. Changing the model on a connected row does
not ask for the key again.

## The credential never enters the container

Model calls go through the platform's inference service under the account's policy. The
account's agent container never holds a provider key or a subscription token: it asks the
platform for a turn, and the platform makes the call. For a subscription that is the only
way it can work at all, since the provider bills the plan only for its own first-party
client, which the platform can present and a container cannot.

## What code sees

| Answer | Meaning | The person does |
|---|---|---|
| `422 · capability_unavailable · no model` | the account has no working model source | **Connect** one under AI Setup |
| `422 · capability_unavailable · no agent containers` | the account's memory cannot run a turn on this deployment | |
| `422 · reason: dormant` | a free-plan account whose free turns are spent | bring a model of its own, or move to a paid plan |
| `429 · rate_limited` | the account's concurrent-turn budget | retry later; the SDK does, twice |
| `add_document` queued, then *Run failed* | the run started, the model call did not | check the row under AI Setup for a failure reason |

An account with no model keeps its memory; searches over what it already learned still
answer, and documents wait unread until a model is set.

## If something looks wrong

| It says | What it means | Do |
|---|---|---|
| *Not connected* on a subscription card | the person has not signed in with that provider | **Connect** and finish the provider's sign-in |
| a failure reason on a key's row | the one test request with that key did not succeed; the key is not used | check the key and the model, **Connect** again |
| everything works except agent turns | the active source is set, the assistant's own model source is not | Home › assistant chip › Model card |
