# Obsidian setup

This page shows how to put the dashboard in your own Obsidian vault. It also lists the problems that can happen and how to fix them.

## What you need

| Item | Version | Note |
| --- | --- | --- |
| Obsidian | 1.13.7 was tested | Dataview 0.5.68 needs Obsidian 0.13.11 or newer. |
| Dataview plugin | 0.5.68 was tested | A community plugin. Obsidian core cannot run code. |

The dashboard was tested on Linux. It was not tested on other versions of Obsidian, on Windows, on macOS, or in the mobile app.

The dashboard needs no other plugin, no account, and no network.

## Steps

1. Open your vault in Obsidian.
2. Open Settings, then Community plugins.
3. If Obsidian shows Restricted mode, press the button to turn on community plugins.
4. Press Browse. Search for `Dataview`. Press Install, then Enable.
5. Open the Dataview settings. Turn on `Enable JavaScript Queries`.
6. Copy all eight files from the `notes/` folder of this repo into your vault. Use any folder. Put all eight files in the same vault.
7. Open the note `Emergency fund dashboard`.
8. Switch to Reading view. The default key is `Ctrl+E`.
9. Check that the Status table shows the sample numbers.
10. Open the note `Emergency fund(log)`. Delete the five sample rows in the Log table. Write your own first row.
11. Open `Emergency fund dashboard` again. Check that the Status table shows your row.

## The eight notes

| Note | Purpose |
| --- | --- |
| `Emergency fund(log)` | The Log table. You write here each day. |
| `Emergency fund dashboard` | The dashboard code. You read here. |
| `Emergency fund settings` | The table of nine numbers. The dashboard reads it. |
| `Emergency fund rules` | The rules of the system, in plain words. |
| `Emergency fund test` | The three questions that decide if an expense is an emergency. |
| `Emergency fund states` | The meaning of Green, Yellow, Orange, and Red. |
| `Emergency fund checklist` | The steps to follow before you take cash out. |
| `Emergency fund how to log` | How to add a row. |

Three notes affect the numbers: `Emergency fund dashboard`, `Emergency fund(log)`, and `Emergency fund settings`. The other five are text for you to read and edit.

## Names that you must keep

- Keep the note names `Emergency fund settings` and `Emergency fund(log)`. The dashboard code finds each note by its name. You can move the notes to another folder.
- Keep the heading `## Log` in the log note. The dashboard reads the rows after this heading.
- Keep the first word of each row in the Settings table. See the next section.

To use another name for a note, change the name in the matching line in the code block of the dashboard note:

```js
const sp = dv.page("Emergency fund settings");
const lp = dv.page("Emergency fund(log)");
```

## How the Settings table is read

The dashboard finds each setting by the start of the label in the first column.

| Label starts with | Meaning |
| --- | --- |
| `Floor` | The lowest level that the fund must keep. |
| `Single` | The single-event limit, in percent of the balance. |
| `Drawdown` | The drawdown limit, in percent of the peak. |
| `Stage 1`, `Stage 2`, `Stage 3` | The three targets. Stage 3 is the long-term target. |
| `Green`, `Yellow`, `Orange` | The fill level at which each state starts, in percent of Stage 3. |

The case of the letters does not matter. The text after the first words is free. You can write `Floor (₹)` or `Floor in dollars`. The value can have a comma, a `%` sign, or a `₹` sign.

## Problems and fixes

| What you see | Cause | Fix |
| --- | --- | --- |
| The code block shows as plain text | Dataview is off, or JavaScript queries are off | Do steps 4 and 5 again. Switch to Reading view. |
| `Note not found: Emergency fund settings` | The settings note is missing or has another name | Copy the note into the vault, or change the name in the code. |
| `Note not found: Emergency fund(log)` | The log note is missing or has another name | Copy the note into the vault, or change the name in the code. |
| `Settings table error ... Missing or wrong value: floor` | The row for the setting is missing or its value is not a number | Check the label and the value in the Settings table. The names in the message are `floor`, `single_limit_pct`, `drawdown_limit_pct`, `stage_1`, `stage_2`, `stage_3`, `green_pct`, `yellow_pct`, and `orange_pct`. |
| `No log lines yet` | The dashboard found no valid row after `## Log` in the log note | Write the date as `YYYY-MM-DD`. Write the total as a number. |
| One day is missing from the numbers | A row has a wrong date or total | Check the row. The dashboard skips a row that it cannot read. |

## Security of JavaScript queries

A Dataview JavaScript query is code. Obsidian runs it with access to your vault. Read the code in a note before you trust it.

The code block in this repo reads two other notes and shows tables. It does not write to your vault. It does not use the network. Check this yourself: the block is about 90 lines in `Emergency fund dashboard`.
