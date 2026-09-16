# Troubleshooting

Look up the word on screen.

## On a memory

| It says | What it means | Do |
|---|---|---|
| **Run** is disabled | the memory has no source yet, or its state is still loading | add a source; wait for *Checking…* to finish |
| *Update now* | a source holds material this memory has not read | press it |
| *Run failed* | the last run did not finish | open **Report**; usually the model source is off or a source needs reauthorization |
| *Nothing yet* | never run | press **Run** |
| *Empty* on the tile | no instruction | Settings › Instruction |
| *Add a source* on the tile | reads nothing yet | Add card |

## On a source

| It says | What it means | Do |
|---|---|---|
| *Queued* / *Syncing…* | fetching raw material | wait; the memory reads it on its next run |
| *Sync failed* | the last fetch failed | **Sync now**; if it repeats, the folder may have moved |
| *Needs reauthorization* | the source's own credential expired | **Reauthorize** |
| *Access revoked* / *Disconnected* | the source no longer grants access | **Reconnect** or remove it |

## On Connect

| It says | What it means | Do |
|---|---|---|
| *Set up* | this app is not connected | pick the tile and follow the steps |
| *Uses nothing yet* (amber) | connected, but no memory switched on | turn a switch on under **Uses** |
| the app cannot see a memory | its switch is off | memory page › **Use in** |
| I removed the app in Claude but it still shows *Connected* | the app did not tell Membase | **Disconnect…** here |

## On Home

| It says | What it means | Do |
|---|---|---|
| *Membase Intelligence is off* | no model source | **AI Setup › Connect** a key or a subscription |
| the reply spins | first reply after a quiet period wakes the assistant | wait; if it does not land, **Retry** |
| *New reply* pill | an answer landed while you scrolled up | click it |
