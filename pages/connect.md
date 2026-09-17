# Connect

Connect is where AI apps get permission to read your memories.

![Connect](../shots/connect-page.png)

1. **The client catalog, by purpose.** *Chat assistants* and *Coding tools*. A dashed tile says
   *Set up*; a solid tile says *Connected* with the time of its last recall and opens the app's
   page.

Picking a dashed tile unfolds the connector URL and the steps for that app:

![Claude's steps](../shots/connect-claude-steps.png)

1. **The steps.** Written for that app's own menus. Follow them in the app; the last one brings
   you back here to approve access and tick the memories the app may read.

## An app's page

Once connected, the tile opens the app's page: the hero (name, *Connected · last recall*), a
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

Under the apps sits the **Developer keys** card: your own credentials for a script, a server or
an SDK. A key reaches the memories you tick, at an access level you choose — **Read**, **Read &
write** (adds facts and documents) or **Manage** (may also delete a document or forget a fact,
and even then only when the call says so explicitly). **New key** makes one and shows the token
once; **Revoke** ends it at once. The walkthrough is
[Use your memory from code](../getting-started/developer-keys.md).

Removing the connector inside Claude or ChatGPT does not tell Membase. To be sure access has
stopped, revoke it here.

## If something looks wrong

Look up the word on screen.

| It says | What it means | Do |
|---|---|---|
| *Set up* | this app is not connected | pick the tile and follow the steps |
| *Uses nothing yet* (amber) | connected, but no memory switched on | turn a switch on under **Uses** |
| the app cannot see a memory | its switch is off | memory page › **Use in** |
| I removed the app in Claude but it still shows *Connected* | the app did not tell Membase | **Disconnect…** here |
| a key's row says *expired* | its token has lapsed | **Revoke** it and make a new one |
| *No developer keys yet* | you have not made one | **New key** on the Developer keys card |

