---
name: dashboard-design
description: Design-reference for building dashboard, overview, analytics, and KPI surfaces — applies a dashboard design system (operational/analytical/strategic types, value→context→trend→detail hierarchy, KPI card anatomy, chart-selection decision tree, semantic color, 8px spacing, zero-state + destructive patterns) for use with any component library. Use when the user says "/dashboard-design", "check the dashboard guide", or when building/redesigning any dashboard, overview, stats, analytics, or metrics surface. Pairs well with a page-layout reference skill for the structural skeleton.
---

# Dashboard Design

Apply a dashboard design system when building data/metrics surfaces, then build it with
**your project's component primitives** — never raw color classes, never a new chart lib without cause.

Distilled from the Dashboard Design Visual Guide (source: https://how-to-dashboard.vercel.app/).
The durable principles live here + in [REFERENCE.md](REFERENCE.md); apply them, don't re-fetch the URL.

## When this fires

- Explicit: "/dashboard-design", "check dashboard guide".
- Proactive: building or redesigning any dashboard, analytics, stats, KPI cards, or "metrics at a glance" view.

## Workflow

1. **Classify the dashboard** (REFERENCE §1): Operational (live, glanceable, refreshing),
   Analytical (explore/compare, denser), or Strategic (periodic, high-level KPIs). The type sets
   density, refresh, and chart choices. State which one you're building before laying anything out.
2. **Establish hierarchy** (REFERENCE §2): every metric is layered **value → context → trend →
   detail**. "A number without context means nothing" — a bare value with no comparison/target is
   the #1 anti-pattern. Decide the layers per metric first.
3. **Lay out the 3-row anatomy** (REFERENCE §3): row 1 = headline KPIs (4–6 cards max), row 2 =
   primary charts/trends, row 3 = detail tables/breakdowns. Size by importance — don't force equal
   sizing across unequal content.
4. **Pick charts by the question, not the data** (REFERENCE §4): run the decision tree. Use the
   project's charting library — see Charts below.
5. **Apply semantic color + 8px spacing** (REFERENCE §5): color carries meaning (up/down,
   good/bad/neutral), never decoration; max 3–4 colors; all spacing on the 8px grid.
6. **Rapid-read test** (REFERENCE §6): can a user extract the top insight in ~5 seconds? If not,
   hierarchy or density is wrong — fix before shipping.

## Build with your primitives

Adapt this table to whatever your project already has. These are the common shadcn/ui mappings:

| Guide concept | shadcn/ui primitive | Notes |
| --- | --- | --- |
| KPI / stat card | A `Card` variant you build once — `value`, `trend`, `subtitle` props | Encode the value→trend→subtitle hierarchy once, reuse everywhere. |
| Card surfaces | `Card`/`CardHeader`/`CardContent` | The base card primitive. |
| Charts | shadcn `chart` (Recharts) | Install on first need: `npx shadcn@latest add chart`. |
| Trend up/down | An icon set (e.g. `ArrowUp`/`ArrowDown`) | Pair with color for accessibility. |
| Spacing | Tailwind scale — multiples of 2 on a 4px base → 8px rhythm: `gap-4`, `p-6`, `space-y-8` | |
| Grid | Tailwind `grid grid-cols-*` | 4–6 KPI columns on desktop, collapse on mobile. |

If your project uses a different component library, map these concepts to your equivalents. The
**design constraints** (hierarchy, sizing, color rules) are not library-specific.

## Hard rules

- **Semantic color via theme vars, not raw palette classes.** Reserve color for meaning: up/down,
  good/warning/bad. Max 3–4 active colors; everything else is neutral. Never use color as the
  *only* signal — pair with icon/arrow/label (accessibility).
- **Do not modify shared UI primitives.** Compose and override from the caller.
- **No new charting dependency** without a deliberate decision — new chart libs add bundle weight.
  Use whatever charting library is already in the project.
- **4–6 KPI cards per view, ≤5 donut/pie slices, ≤3–4 colors, bar axes start at zero.** These
  constraints are the point — don't relax them to fit more in.
- Keep dashboard components **generic and theme-driven** — restyle via CSS vars, not by editing
  component internals.

## See also

- [REFERENCE.md](REFERENCE.md) — full distilled guide: types, hierarchy, anatomy, KPI card types,
  chart decision tree, color/spacing systems, zero-state + destructive patterns, industry metrics.
- Source: https://how-to-dashboard.vercel.app/
