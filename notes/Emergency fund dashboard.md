Part of the emergency fund system. Add the daily rows in [[Emergency fund(log)]]. See [[Emergency fund how to log]] for the setup.

## Dashboard

![[Emergency fund chart]]

```dataviewjs
const L = dv.luxon.DateTime;
const money = n => "₹" + Math.round(n).toLocaleString("en-IN");
const pct = n => Math.round(n) + "%";

// Read the table in the note "Emergency fund settings": Setting | Value. A row matches by the start of its label.
const sp = dv.page("Emergency fund settings");
if (!sp) return dv.paragraph("Note not found: Emergency fund settings");
const rows = (await dv.io.load(sp.file.path)).split("\n").map(l => l.split("|").map(s => s.trim()));
const find = re => parseFloat(((rows.find(c => re.test(c[1] ?? "")) ?? [])[2] ?? "").replace(/[₹,%\s]/g, ""));
const p = {
  floor: find(/^floor/i),
  single_limit_pct: find(/^single/i),
  drawdown_limit_pct: find(/^drawdown/i),
  stage_1: find(/^stage 1/i), stage_2: find(/^stage 2/i), stage_3: find(/^stage 3/i),
  green_pct: find(/^green/i), yellow_pct: find(/^yellow/i), orange_pct: find(/^orange/i),
};
const bad = Object.keys(p).filter(k => isNaN(p[k]));
if (bad.length) return dv.paragraph("Settings table error in Emergency fund settings. Missing or wrong value: " + bad.join(", "));

// Read the table under "## Log" in the note "Emergency fund(log)": Date | Total | Note. The last row for a date wins.
// Header, separator, and empty rows fail the checks and are skipped.
const lp = dv.page("Emergency fund(log)");
if (!lp) return dv.paragraph("Note not found: Emergency fund(log)");
const raw = await dv.io.load(lp.file.path);
const body = raw.split(/^## Log[ \t]*$/m)[1] ?? "";
const byDate = new Map();
for (const line of body.split("\n")) {
  const [, date, total, note] = line.split("|").map(s => s.trim());
  if (/^\d{4}-\d{2}-\d{2}$/.test(date ?? "") && /^₹?\s*\d[\d,]*(\.\d+)?$/.test(total ?? ""))
    byDate.set(date, { amount: Number(total.replace(/[₹,\s]/g, "")), note: note ?? "" });
}
const log = [...byDate].sort(([a], [b]) => a.localeCompare(b)).map(([date, v]) => ({ date, ...v }));
if (!log.length) return dv.paragraph("No log lines yet. Add one under the Log heading in Emergency fund(log).");

const now = log.at(-1);
const today = L.now().startOf("day");
const gap = Math.round(today.diff(L.fromISO(now.date), "days").days);

// A fall between two log lines is a withdrawal. A rise is a deposit.
const events = log.slice(1).map((e, i) => ({ date: e.date, note: e.note, before: log[i].amount, delta: e.amount - log[i].amount }));

const peak = Math.max(...log.map(e => e.amount)); // reference level for the drawdown
const owed = peak - now.amount;
const drawdown = peak ? (owed / peak) * 100 : 0;

const ofTarget = (now.amount / p.stage_3) * 100;
const [state, order] =
  ofTarget >= p.green_pct ? ["🟢 GREEN", "Normal financial activity."] :
  ofTarget >= p.yellow_pct ? ["🟡 YELLOW", "Replenishment comes before discretionary purchases."] :
  ofTarget >= p.orange_pct ? ["🟠 ORANGE", "No unnecessary withdrawals. Replenish hard."] :
  ["🔴 RED", "Rebuilding the fund is the main money goal."];

const limitFor = bal => Math.max(0, Math.min((p.single_limit_pct / 100) * bal, bal - p.floor));
const maxOut = limitFor(now.amount);
const next = [p.stage_1, p.stage_2, p.stage_3].find(s => s > now.amount);

const span = Math.min(30, Math.max(1, Math.round(today.diff(L.fromISO(log[0].date), "days").days)));
const since = today.minus({ days: span }).toISODate();
const perDay = events.filter(e => e.delta > 0 && e.date > since).reduce((s, e) => s + e.delta, 0) / span;
const days = next && perDay > 0 ? Math.ceil((next - now.amount) / perDay) : null;

const ym = today.toFormat("yyyy-MM");
const month = events.filter(e => e.date.startsWith(ym));
const mIn = month.filter(e => e.delta > 0).reduce((s, e) => s + e.delta, 0);
const mOut = -month.filter(e => e.delta < 0).reduce((s, e) => s + e.delta, 0);

dv.header(3, "Status");
dv.table(["Item", "Now"], [
  ["Balance", `${money(now.amount)} on ${now.date}`],
  ["Log", gap <= 0 ? "Up to date" : gap === 1 ? "Last line is from yesterday" : `MISSING: last line is ${gap} days old`],
  ["State", `${state} (${pct(ofTarget)} of ${money(p.stage_3)})`],
  ["Rule for this state", order],
  ["Max withdrawal now", maxOut > 0 ? money(maxOut) : "₹0 (no discretionary withdrawal)"],
  ["Floor", now.amount >= p.floor ? `${money(p.floor)} (kept)` : `${money(p.floor)} (${money(p.floor - now.amount)} below)`],
  ["Next stage", next ? `${money(next)}, ${money(next - now.amount)} to go` : "All stages reached"],
  ["Replenishment owed", `${money(owed)} (${pct(drawdown)} below the peak of ${money(peak)})`],
  ["Drawdown limit", drawdown >= p.drawdown_limit_pct ? `BREACH: replenish first (limit ${p.drawdown_limit_pct}%)` : `OK (limit ${p.drawdown_limit_pct}%)`],
  ["This month", `${money(mIn)} added, ${money(mOut)} taken`],
  [`Pace (last ${span} days)`, `${money(perDay)} per day`],
  ["Next stage at this pace", !next ? "Done" : days ? `${days} days (${today.plus({ days }).toISODate()})` : "No deposits yet"],
]);

dv.header(3, "Withdrawals");
const out = events.filter(e => e.delta < 0).map(e => {
  const amt = -e.delta, limit = limitFor(e.before);
  return [e.date, money(amt), pct((amt / e.before) * 100), money(limit), amt <= limit ? "OK" : "OVER LIMIT", e.note];
});
if (out.length) dv.table(["Date", "Taken", "Share of fund", "Limit then", "Verdict", "Note"], out);
else dv.paragraph("No withdrawals in the log.");
```
