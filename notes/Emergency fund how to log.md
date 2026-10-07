Part of [[Emergency fund(log)]]. This note shows how to add a log row and how to make the dashboard work.

## Log a day

Add one row each day at the bottom of the Log table in [[Emergency fund(log)]]. Write the date in the first column. Write the total cash in hand in the second column. The note is optional.

To add a row, put the cursor in the last cell and press `Enter`.

- Write the date as `YYYY-MM-DD`.
- Write the total as a number. A comma or the `₹` sign is allowed.
- Do not write anything after the Log table.
- A total that falls is a withdrawal. A total that rises is a deposit.
- If I fix a wrong row, I add a new row for the same date. The last row for that date wins.

## Setup (one time)

1. Open Settings, then Community plugins, then Browse.
2. Install and enable the plugin Dataview.
3. In the Dataview settings, turn on "Enable JavaScript Queries".

Obsidian core cannot run code. Without Dataview, the dashboard stays empty.

The dashboard reads two tables: the Log in [[Emergency fund(log)]] and the numbers in [[Emergency fund settings]]. Change a number there and the dashboard changes.
