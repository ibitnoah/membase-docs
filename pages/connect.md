# Connect

Connect is where an AI gets permission to read your memories. There are two ways to connect an
AI, and the page has a tab for each.

![Connect](../shots/connect-page.png)

1. **The tabs.** **MCP** is for an AI app; **Skills** is for an AI that reads pages and runs
   commands. Each tab holds everything its way needs.
2. **The address** (MCP tab). Copy it into the app's connector or MCP settings; the app opens
   your browser and you approve it on the consent card, ticking the memories it may read. An
   app connected this way can only read.
3. **The client catalog, by purpose.** *Chat assistants* and *Coding tools*. A dashed tile says
   *Set up* and unfolds that app's steps; a solid tile says *Authorized* and opens the app's
   page.

![The Skills tab](../shots/connect-start.png)

The **Skills** tab keeps setup in three steps: create a key, paste the instruction, and let
your AI confirm which memories it can use. It works with coding assistants such as Claude
Code, Codex and Cursor that can read pages and run commands.

1. **Create key.** Choose the access and memories your AI may use.
2. **Already have a key?** Expand this to copy an installation instruction without creating
   another key. Your AI installs the skill, then asks for your existing key.
3. **Download skill** gets the complete folder for a manual installation. **Setup guide**
   opens the detailed instructions.
4. **Manage keys** opens your developer keys, where you can change access or revoke a key.

## The skill way, step by step

1. Press **Create key**. The **A key for your AI** dialog starts with name *my AI* and access
   **Read & write**. Choose the memories it may reach and press **Create key**.
2. Copy the instruction on the next screen. It includes the key, which is shown only once.
3. Paste it to your AI. It installs the skill, stores the key, and reports which memories it
   can use. If none are selected, open **Manage keys**, choose the key, and update its reach.

The skill uses the API with your developer key. If the AI already has Membase MCP tools,
it uses those tools and keeps the skill for its rules.

Manual MCP configuration is under **MCP › Manual configuration**. API documentation is under
**Manage keys › API reference**.

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

The Skills tab’s **Manage keys** link opens your credentials for an AI, a script, a server
or an SDK. Each key has its own row —
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
| my AI installed the skill but says it has no access | the skill needs a developer key | Skills tab › **Create key**, and paste the sentence to the AI |
| a key's row says *Expired* | its token has lapsed | **⋯ › Create a similar key** on the Developer keys page |
| I already have a key | you can reuse it | Skills tab › **Already have a key?** |

