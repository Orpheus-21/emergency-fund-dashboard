```dataviewjs
const L = dv.luxon.DateTime;
const money = n => "₹" + Math.round(n).toLocaleString("en-IN");
const short = n => "₹" + (n >= 1000 ? +(n / 1000).toFixed(1) + "k" : Math.round(n));
const r = n => Math.round(n * 10) / 10;

// Read the floor and the stages from the note "Emergency fund settings", as the dashboard does.
const sp = dv.page("Emergency fund settings");
if (!sp) return dv.paragraph("Note not found: Emergency fund settings");
const rows = (await dv.io.load(sp.file.path)).split("\n").map(l => l.split("|").map(s => s.trim()));
const find = re => parseFloat(((rows.find(c => re.test(c[1] ?? "")) ?? [])[2] ?? "").replace(/[₹,%\s]/g, ""));
const levels = [["Floor", find(/^floor/i)], ["Stage 1", find(/^stage 1/i)], ["Stage 2", find(/^stage 2/i)], ["Stage 3", find(/^stage 3/i)]];
if (levels.some(([, v]) => isNaN(v))) return dv.paragraph("Settings table error in Emergency fund settings.");

// Read the table under "## Log" in the note "Emergency fund(log)": Date | Total | Note. The last row for a date wins.
const lp = dv.page("Emergency fund(log)");
if (!lp) return dv.paragraph("Note not found: Emergency fund(log)");
const body = (await dv.io.load(lp.file.path)).split(/^## Log[ \t]*$/m)[1] ?? "";
const byDate = new Map();
for (const line of body.split("\n")) {
  const [, date, total] = line.split("|").map(s => s.trim());
  if (/^\d{4}-\d{2}-\d{2}$/.test(date ?? "") && /^₹?\s*\d[\d,]*(\.\d+)?$/.test(total ?? ""))
    byDate.set(date, Number(total.replace(/[₹,\s]/g, "")));
}
const log = [...byDate].sort(([a], [b]) => a.localeCompare(b)).map(([date, amount]) => ({ date, amount }));
if (!log.length) return dv.paragraph("No log lines yet. Add one under the Log heading in Emergency fund(log).");

// The picture is 680 by 280 units. The vertical axis ends at the first target above the peak.
const W = 680, H = 280, pl = 58, pr = 96, pt = 22, pb = 34;
const d0 = L.fromISO(log[0].date);
const span = Math.max(1, L.fromISO(log.at(-1).date).diff(d0, "days").days);
const peak = Math.max(...log.map(e => e.amount));
const ymax = (levels.map(([, v]) => v).filter(v => v > peak).sort((a, b) => a - b)[0] ?? peak * 1.1) || 1;
const x = d => r(log.length > 1 ? pl + (L.fromISO(d).diff(d0, "days").days / span) * (W - pl - pr) : (pl + W - pr) / 2);
const y = v => r(H - pb - (v / ymax) * (H - pt - pb));

// Dashed lines for the floor and the stages. Levels with the same value share one line.
const byValue = new Map();
for (const [name, v] of levels) if (v <= ymax) byValue.set(v, [...(byValue.get(v) ?? []), name]);
const guides = [...byValue].map(([v, names]) =>
  `<line x1="${pl}" x2="${W - pr}" y1="${y(v)}" y2="${y(v)}" style="stroke:var(--text-faint);stroke-dasharray:4 4"/>` +
  `<text x="${pl - 6}" y="${y(v) + 4}" text-anchor="end" style="fill:var(--text-muted)">${short(v)}</text>` +
  `<text x="${W - pr + 6}" y="${y(v) + 4}" style="fill:var(--text-faint)">${names.join(" / ")}</text>`).join("");

const k = Math.min(4, Math.round(span));
const ticks = log.length > 1 ? Array.from({ length: k + 1 }, (_, i) => d0.plus({ days: Math.round((i * span) / k) }).toISODate()) : [log[0].date];
const axis =
  `<line x1="${pl}" x2="${W - pr}" y1="${y(0)}" y2="${y(0)}" style="stroke:var(--background-modifier-border)"/>` +
  `<text x="${pl - 6}" y="${y(0) + 4}" text-anchor="end" style="fill:var(--text-muted)">₹0</text>` +
  ticks.map(d => `<text x="${x(d)}" y="${H - 10}" text-anchor="middle" style="fill:var(--text-muted)">${L.fromISO(d).toFormat("d LLL")}</text>`).join("");

const pts = log.map(e => `${x(e.date)},${y(e.amount)}`).join(" ");
const area = `${x(log[0].date)},${y(0)} ${pts} ${x(log.at(-1).date)},${y(0)}`;
const dots = log.map((e, i) => {
  const drop = i > 0 && e.amount < log[i - 1].amount;
  return `<circle cx="${x(e.date)}" cy="${y(e.amount)}" r="${i === log.length - 1 ? 5 : 3}" style="fill:${drop ? "var(--text-error)" : "var(--interactive-accent)"}"><title>${e.date}: ${money(e.amount)}</title></circle>`;
}).join("");
const now = log.at(-1);
const label = `<text x="${x(now.date)}" y="${y(now.amount) - 11}" text-anchor="${x(now.date) < W / 2 ? "start" : "end"}" style="fill:var(--text-normal);font-weight:600;paint-order:stroke;stroke:var(--background-primary);stroke-width:4px">${money(now.amount)}</text>`;

dv.container.createDiv().innerHTML =
  `<svg viewBox="0 0 ${W} ${H}" width="100%" role="img" aria-label="Emergency fund balance by date" style="max-width:${W}px;font:11px var(--font-interface)">` +
  `${guides}${axis}<polygon points="${area}" style="fill:var(--interactive-accent);opacity:0.12"/>` +
  `<polyline points="${pts}" style="fill:none;stroke:var(--interactive-accent);stroke-width:2"/>${dots}${label}</svg>`;
dv.paragraph("Balance by date. Dashed lines: the floor and the stages. Red dots: a day when the total fell.");
```
