# Connect

Connect is where an AI gets permission to read your memories. Every AI gets in one of two ways:
an **app you approve** in your browser, or a **developer key you hold**. The page is laid out in
that order — the two ways to reach the door at the top, the approvals below.

![Connect](../shots/connect-page.png)

1. **The address.** The MCP server every client uses. Copy it into an app's connector or MCP
   settings when you set the app up by hand (the tiles below give the steps).
2. **Or tell your AI.** One sentence — *Install Membase from https://www.app.membase.io/skill* —
   for an AI that can read a page and run commands (Claude Code, Codex, Cursor, Windsurf…).
   Paste it into the AI: it installs the Membase skill and the server itself, then asks you to
   sign in or for a developer key. Neither the address nor the sentence grants anything; the
   AI still ends up on one of the two approvals below.
3. **The client catalog, by purpose.** *Chat assistants* and *Coding tools*. A dashed tile says
   *Set up*; a solid tile says *Authorized* with the time of its last use and opens the app's
   page. *Your own code* holds **Developer keys**.

![Start here](../shots/connect-start.png)

**Setup files** (3), at the end of the second row, unfolds what the sentence fetches, for hands
that cannot run commands: the skill folder as a zip (for a Claude.ai upload or `~/.claude/skills/`),
the `mcp.json` server block Cursor, Windsurf and Claude Desktop read, the VS Code and Codex
snippets, the Claude Code command (with the developer-key variant), and the public API
reference.

## Tell your AI

Paste the sentence into your AI. What happens next is the AI's work, but it always ends in the
same two places:

1. The AI reads the page at that address — the skill's own SKILL.md, which begins with install
   steps — fetches the skill folder (SKILL.md and five references) and adds the MCP server for
   its client.
2. It has no access yet, and says so. It asks you which way you want to grant it:
   - **Sign in** — the AI runs its client's MCP login, your browser opens the approval below,
     you tick the memories it may read. It will be read-only.
   - **Developer key** — you make one under **Developer keys** and paste the token to the AI.
     You pick its access level.
3. It makes one call to list the memories it may use and tells you the result. An empty list
   means no memory is switched on for it yet: open the memory and switch it on under **Use in**.

## Set an app up by hand

Picking a dashed tile unfolds the connector URL and the steps for that app:

![Claude's steps](../shots/connect-claude-steps.png)

1. **The steps.** Written for that app's own menus. Follow them in the app; the last one brings
   you back here to approve access and tick the memories the app may read.

## An app's page

Once connected, the tile opens the app's page: the hero (name, *Authorized · last activity*), a
**Uses** card with one row per memory and its switch, and, when there is more than one approval,
an **Approvals** card with a **Revoke** per credential. **Disconnect…** in the hero revokes every
approval at once.

> Connecting is the account's, once. Which memories an app uses is the memory's, and the switch on
> the app's page and the switch on the memory's *Use in* card are the same switch.

## What the app can do

A connected app gets two tools: list the memories it may use, and search them. It cannot see
memories you have not switched on for it, and it cannot tell that they exist. Turning a switch
off takes effect on the app's very next question. If you ticked **Let it know about you** when
approving, it also gets your profile — the standing facts your assistant keeps about you and
what changed recently. An app can never add, delete or forget anything.

## Developer keys

![Developer keys](../shots/connect-developer-keys.png)

Under the apps, the **Your own code** group holds the **Developer keys** tile: your own
credentials for a script, a server or an SDK. It opens a page of its own, one key per row —
name and hint, access (**Read**, **Read & write** or **Full access**, which may also delete a
document or forget a fact, and even then only when the call says so explicitly), reach, expiry
and last use. **Create key** makes one and shows the token once; a key's page changes its
access and reach in place, shows what it has been doing, and rotates or revokes it. The
walkthrough is [Use your memory from code](../getting-started/developer-keys.md).

Removing the connector inside Claude or ChatGPT does not tell Membase. To be sure access has
stopped, open the app's page here and **Disconnect…**.

## If something looks wrong

Look up the word on screen.

| It says | What it means | Do |
|---|---|---|
| *Set up* | this app is not connected | pick the tile and follow the steps |
| *Uses nothing yet* (amber) | connected, but no memory switched on | turn a switch on under **Uses** |
| the app cannot see a memory | its switch is off | memory page › **Use in** |
| I removed the app in Claude but it still shows *Authorized* | the app did not tell Membase | **Disconnect…** on the app's page |
| *The MCP address is unavailable* | the page could not ask the server for it | reload; the sentence for your AI still works |
| my AI installed the skill but says it has no access | installing grants nothing | answer its question: sign in, or give it a developer key |
| a key's row says *Expired* | its token has lapsed | **⋯ › Create a similar key** on the Developer keys page |
| the Developer keys tile says *For scripts and SDKs* | you have no active key | open it and press **Create key** |

