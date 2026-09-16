# AI Setup

AI Setup is where the account's model comes from.

![AI Setup](../shots/ai-setup-page.png)

Two ways in:

- **Bring your own key** for a provider (OpenAI, Anthropic, Gemini, DeepSeek, Kimi, Qwen,
  OpenRouter, or any OpenAI-compatible endpoint). Press **Connect**, paste the key, pick a model
  from the list the key returns. A key with no model is not saved.
- **A Claude or ChatGPT subscription.** Press **Connect** on the subscription card and sign in
  with that provider. Turns are then billed to the subscription.

The **Connected** tab lists what you have set up; one of them is the account's **active**
source. The active source answers everything that is not an agent turn. Which source *the
assistant* runs on is the assistant's own field, in its settings on Home.

## Verify

Each connection is verified with one short real request before it is used. A key that fails
verification is never used, and the row says why.
