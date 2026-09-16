# Reach your assistant on Telegram

Your assistant can live in a Telegram chat as well as on Home. The same memory answers in both
places, and anything a schedule produces is delivered to that chat.

## Connect

1. On Home, look at the foot of the conversation rail, under **Remote**. Press
   **Connect Telegram**.
2. In the dialog, keep **Scan a QR** and press **Generate QR**.

![Connect Telegram](../shots/telegram-dialog.png)

3. Scan the QR with your phone's camera, or press **Copy link** and open the link in Telegram.
   Either way Telegram opens the Membase bot with a one-time code already filled in; send it.
4. Back in the dialog press **I scanned it — refresh**. The chat appears under
   **Connected chats**, and the **Remote** row on Home now opens that chat.

The code is single-use, valid for 15 minutes and tied to your account. Nothing is connected
until the bot actually receives it from your phone.

### Your own bot

If you would rather not share the Membase bot, pick **Use my own bot**. Create a bot with
@BotFather in Telegram (send it `/newbot`), paste the bot token the dialog asks for, then generate
the QR the same way. The optional **Webhook secret token** is for operators who run their own
webhook and can be left empty.

## What the chat can do

- **Talk to the assistant.** Ask anything you would ask on Home. It reads the same memories and
  keeps the same instructions.
- **Receive scheduled results.** When a memory or agent on a schedule finishes a run, its result
  is sent to this chat. There is nothing to configure.
- **Read it on Home.** The **Remote** row opens a read-only view of the Telegram chat, so what
  you said on your phone is there when you are back at your desk.

The Telegram chat is its own conversation. It does not appear in the day-grouped list on Home,
and Home's conversations do not appear in Telegram; the memory is what they share.

## Disconnect

Open the dialog again from the **Remote** row (the ⚙ beside it) and press **Delete** next to the
chat under **Connected chats**. The bot stops answering that chat at once. Scheduled results
stop being delivered there too, and stay on the Schedules and Activity pages.
