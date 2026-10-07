# The formulas

This page explains every number on the dashboard. Each number has a name, a plain meaning, a formula, and an example.

All examples use the default settings:

| Setting | Default |
| --- | --- |
| Floor | 10,000 |
| Single-event limit | 20% of the balance |
| Drawdown limit | 30% of the peak balance |
| Stage 1, Stage 2, Stage 3 | 10,000, 25,000, 50,000 |
| Green, Yellow, Orange start at | 100%, 80%, 60% of Stage 3 |

The real dashboard code gave the results that this page shows. The only exception is the example about the pace in section 9. That example is calculated by hand from the formula.

## The idea in one paragraph

A trader does not risk the whole account on one trade. The trader sets a loss limit and keeps to it. This system gives the emergency fund the same kind of limits. You write one number each day: the total cash in the fund. The dashboard compares that number with your limits. The dashboard then tells you how much you can take out, how much you owe the fund, and what to do next.

## What the dashboard reads

The dashboard reads two tables and nothing else.

1. The **Log**: one row for each day, with the date and the total in the fund.
2. The **Settings**: nine numbers that you can change.

All formulas below use only these two tables.

## 1. Balance

The balance is the total in the last row of the Log. The last row is the row with the latest date.

```
Balance = total of the row with the latest date
```

Example: the Log ends with `2026-10-05 | 19,300`. The balance is 19,300.

If two rows have the same date, the row that is lower in the note wins. Use this to correct a mistake.

## 2. Peak

The peak is the highest total that any row of the Log shows.

```
Peak = highest total in the Log
```

Example: the totals are 20,000, 20,500, 21,000, 19,000, 19,300. The peak is 21,000.

The peak is the level that you must return to after a withdrawal.

## 3. Replenishment owed

This is the amount that you took out and did not put back.

```
Owed = Peak - Balance
```

Example: the peak is 25,000 and the balance is 21,000. The fund owes 4,000.

When you have no debt, the peak and the balance are equal. The owed amount is 0.

## 4. Drawdown and the BREACH alarm

The drawdown is the owed amount as a percent of the peak.

```
Drawdown % = Owed / Peak x 100
```

The dashboard shows `BREACH` when the drawdown is equal to or more than the drawdown limit.

```
BREACH when Drawdown % >= Drawdown limit
```

| Peak | Balance | Owed | Drawdown | Result |
| --- | --- | --- | --- | --- |
| 50,000 | 35,001 | 14,999 | 29.998% | OK |
| 50,000 | 35,000 | 15,000 | 30% | BREACH |

A BREACH means: put money back into the fund before you do anything else with spare money.

## 5. Fill level and state

The fill level shows how full the fund is. It compares the balance with Stage 3, the long-term target.

```
Fill % = Balance / Stage 3 x 100
```

The state comes from the fill level:

| State | Fill level | With default settings | What to do |
| --- | --- | --- | --- |
| Green | 100% or more | 50,000 or more | Live as normal. |
| Yellow | 80% to 99.99% | 40,000 to 49,999 | Refill the fund before you buy things you want. |
| Orange | 60% to 79.99% | 30,000 to 39,999 | Take out nothing that is not needed. Refill hard. |
| Red | Less than 60% | Less than 30,000 | Rebuilding the fund is your main money goal. |

The state uses the exact fill level. The status table shows the fill level rounded to a whole number. For a balance of 29,999 the table shows `60%`, and the state is Red. The exact value is 59.998%.

## 6. Maximum withdrawal

This number answers the question: how much can I take out now?

```
Max withdrawal = the larger of 0 and (the smaller of
                   (Single-event limit % x Balance)
                   and
                   (Balance - Floor))
```

In plain words:

1. Calculate the single-event limit: 20% of the balance.
2. Calculate the room above the floor: the balance minus the floor.
3. Take the smaller of the two numbers.
4. If the result is below 0, use 0.

The first number stops one event from taking too much. The second number protects the floor.

| Balance | 20% of balance | Balance minus floor | Max withdrawal | Why |
| --- | --- | --- | --- | --- |
| 8,700 | 1,740 | -1,300 | 0 | The balance is below the floor. |
| 12,000 | 2,400 | 2,000 | 2,000 | The floor is the limit. |
| 15,000 | 3,000 | 5,000 | 3,000 | The 20% rule is the limit. |
| 25,000 | 5,000 | 15,000 | 5,000 | The 20% rule is the limit. |
| 50,000 | 10,000 | 40,000 | 10,000 | The 20% rule is the limit. |

The dashboard shows this number in every state. The state tells you if a withdrawal is wise. The number tells you the most that the rules allow.

## 7. Withdrawals and deposits

You never log a withdrawal. You log the total. The dashboard finds the change between two rows.

```
Change = total of this row - total of the row before
```

- A change below 0 is a **withdrawal**. The amount taken is the change without its minus sign.
- A change above 0 is a **deposit**.
- The first row has no row before it. It is neither a withdrawal nor a deposit.

## 8. Check of each withdrawal

The dashboard checks every past withdrawal against the same rule as section 6. It uses the balance **before** the withdrawal, which is the total of the row before.

```
Limit then = max withdrawal, calculated from the balance before
Share      = Taken / balance before x 100
Verdict    = OK when Taken <= Limit then, else OVER LIMIT
```

Example: the balance was 25,000. You took out 4,000. The limit then was 5,000. The share is 16%. The verdict is OK.

Example: the balance was 8,700, which is below the floor. You took out 700. The limit then was 0. The verdict is OVER LIMIT.

OVER LIMIT is not forbidden. The rules allow one exception: a real emergency. The mark makes you see the event, and the debt tells you what to repay.

## 9. Next stage and the pace

The next stage is the first stage that is higher than the balance.

```
To go = Next stage - Balance
```

Example: the balance is 19,300. The next stage is 25,000. To go is 5,700.

When the balance is at or above Stage 3, the dashboard shows `All stages reached`.

The pace is how much you add each day. The dashboard uses the days since your first Log row, with a maximum of 30 days and a minimum of 1 day.

```
Window days  = days from the first Log row to today (at least 1, at most 30)
Added        = sum of all deposits in that window
Pace         = Added / Window days
Days to stage = round up (To go / Pace)
```

Example by hand: you added 3,000 in 30 days. The pace is 100 per day. The To go amount is 1,300. Days to stage is 13.

Withdrawals do not lower the pace. Only deposits count.

## 10. This month

This month is the calendar month of today's date. The dashboard adds up all deposits and all withdrawals whose row date is in that month.

## 11. Log check

The log check counts the days from the date of the last row to today.

| Days | The dashboard shows |
| --- | --- |
| 0 or less | Up to date |
| 1 | Last line is from yesterday |
| 2 or more | MISSING and the number of days |

## Limits of this model

These limits are true for the current code.

- **Net change only.** The dashboard sees only the difference between two rows. If you take out 500 and put in 500 on the same day, the dashboard sees no change.
- **The peak never resets.** The peak is the highest total in the whole Log. To start a new peak, delete the old rows that hold the high totals.
- **Missing days.** If you skip days, the dashboard puts the whole change on the date of the next row. This changes the month totals and the pace.
- **One currency.** The numbers have no unit. The display adds the `₹` sign. See the Configuration section of the README to change it.
- **Defaults are examples.** The default numbers are a starting point. They are not financial advice. Choose numbers that fit your life.
