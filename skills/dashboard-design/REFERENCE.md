# Dashboard Design — Reference

Distilled from the Dashboard Design Visual Guide (https://how-to-dashboard.vercel.app/).
Apply these principles; the live URL is the source, not a runtime dependency.

## §1 — Dashboard types (classify first)

| Type | Purpose | Audience | Density | Refresh | Chart bias |
| --- | --- | --- | --- | --- | --- |
| **Operational** | Monitor live state, react now | operators on shift | high, glanceable | real-time / frequent | status tiles, live line/area, sparklines |
| **Analytical** | Explore, compare, find why | analysts | highest, interactive | on-demand | bar comparisons, scatter, tables, filters |
| **Strategic** | Track goals vs targets | execs | low, summarized | periodic (wk/mo) | big KPIs, trend lines, target gauges |

The type dictates density, refresh cadence, and which charts make sense. State it before laying out.

## §2 — Information hierarchy (the core idea)

Every metric is layered. A bare number is meaningless:

1. **Value** — the number itself (`$48.2K`).
2. **Context** — vs what? (target, prior period, benchmark — `vs $45K goal`).
3. **Trend** — direction + magnitude (`▲ 12% MoM`).
4. **Detail** — drill path / breakdown (link, sparkline, expand).

**#1 anti-pattern: value with no context or comparison.** If you can't add context/trend, question
whether the metric belongs on the dashboard.

## §3 — Anatomy: the 3-row layout

- **Row 1 — Headline KPIs.** 4–6 cards max. Most important metric top-left (F-pattern reading).
- **Row 2 — Primary trends/charts.** The "why" behind row 1. Larger = more important.
- **Row 3 — Detail.** Tables, breakdowns, recent activity, drill-downs.

Size communicates importance. **Never force equal sizing** across elements of unequal weight.
Group related elements by proximity; don't let unrelated tiles sit adjacent (false grouping).

## §4 — Chart selection (question → chart)

Pick by the **question being asked**, not by what data you happen to have.

| Question | Chart | shadcn `chart` variant |
| --- | --- | --- |
| How did X change over time? | line / area | `ChartLine`, `ChartArea` |
| How do categories compare? | bar (horizontal if labels long) | `ChartBar` |
| What's the part-to-whole split? | donut (≤5 slices) | `ChartPie` (donut) |
| Is there a relationship/cluster? | scatter | `ScatterChart` |
| What are the exact values / many dims? | table | `Table` / DataTable |
| Single metric vs target? | KPI card + gauge/progress | stat card + `Progress` |

Rules: **bar axes start at zero** (truncated axes lie); donut/pie **≤5 slices** (else bar);
prefer a table when precision matters more than shape.

If using shadcn/ui, add the chart component on first need:
```bash
npx shadcn@latest add chart
```
shadcn `chart` is Recharts-based — it adds bundle weight, so install deliberately.

## §5 — Color + spacing systems

**Color = meaning, never decoration:**
- Reserve color for semantics: up/down, good/warning/bad, category encoding.
- Max **3–4 colors** in active use. Everything else is neutral.
- Use the **theme's semantic CSS vars** (success/warning/destructive/primary), not raw palette
  classes like `green-500`/`red-500`.
- Color must never be the *only* signal (accessibility) — pair with icon/arrow/label.

**Spacing = 8px rhythm:** all gaps/padding on multiples of 8 (Tailwind: `gap-4`=16, `p-6`=24,
`space-y-8`=32). Consistent positioning across cards reduces cognitive load on repeated daily use.

## §6 — Rapid-read validation

Before shipping, the **5-second test**: can the target user extract the single most important
insight in ~5 seconds? If not, the hierarchy is flat or the view is too dense. Fix by: promoting
the headline metric (size/position), cutting tiles to 4–6, removing decorative color, or moving
detail to row 3 / a drill-down.

## §7 — Platform (tool-like) dashboards

For internal/operator tool surfaces: optimize for **density + speed of repeated use**, not
first-impression polish. Consistent tile positions across pages, tighter spacing (still 8px grid),
minimal chrome, keyboard-reachable. Favor tables + compact stat rows over big hero charts.

## §8 — Zero-state + destructive patterns

- **Zero/empty state:** never show an empty grid. Explain what will appear, why it's empty, and the
  one action to populate it (CTA). Follow your project's established empty-state conventions.
- **Destructive actions on a dashboard:** confirm via a modal dialog (never `window.confirm`); for
  high-risk actions, require type-to-confirm. Keep destructive controls out of the glance path —
  don't put a delete next to a KPI.

## §9 — Industry KPI starters (reference)

Useful when building a client-facing or vertical-specific dashboard:
- **E-commerce:** revenue, AOV, conversion rate, cart abandonment, repeat-purchase rate.
- **SaaS:** MRR/ARR, churn, NRR, activation rate, DAU/MAU.
- **Healthcare:** patient volume, wait time, readmission rate, utilization.
- **Finance:** burn, runway, gross margin, AR aging, cash position.

Pick 4–6 for row 1; the rest are row-3 detail. Always pair each with context + trend (§2).
