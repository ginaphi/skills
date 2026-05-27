---
name: smoke-test
description: Browser smoke test via Chrome DevTools MCP — tiny always-on baseline (app loads, zero console errors) plus session-derived checks built from `git diff` and conversation context. Teaches the accessibility-tree targeting pattern for Chrome DevTools MCP. Use when the user says "/smoke-test", "smoke test this", "check the browser", or "verify what we just built works".
---

# Smoke Test

The job is to **catch crashes in what we just shipped**, not to execute a fixed script. A tiny baseline ensures dumb regressions surface; a session-derived plan targets exactly what changed.

## Prerequisites

Dev server must be running. Set your app URL:

```
APP_URL=http://localhost:3000   # adjust to your dev server
```

If the server isn't running, start it before proceeding.

## Element targeting

Chrome DevTools MCP is **accessibility-tree-driven**. Every `click` and `fill` call takes a `uid` from a snapshot — not a CSS selector.

Pattern for every interaction:

1. Call `take_snapshot` to get the current a11y tree with UIDs.
2. Locate the target by **role + accessible name** (e.g. `button "Submit"`, `textbox "Email"`).
3. Pass that `uid` to `click` or `fill`.

```
# Reliable — found by role + label
button "Submit"    uid: @e42   → click(@e42)
textbox "Email"    uid: @e17   → fill(@e17, "user@example.com")

# Ambiguous — unlabeled, can't distinguish from siblings
button             uid: @e11   # which one?
```

When a required element is missing from the snapshot or appears without an accessible name, treat it as an accessibility gap: note it in the report (`[gap] button at X has no label`) and proceed rather than silently skipping the check.

**`wait_for` uses visible text** — state changes that update only color or icon without changing visible text will time out. Flag color-only status indicators as a gap if encountered.

## Step 0 — Run tests first (optional)

Before opening the browser, run your project's test suite scoped to changed files. A browser pass on logic that's already red is wasted. Skip this step if your pipeline already ran tests upstream.

## Step 1 — Baseline (always, 2 checks)

Always-on regardless of what changed. Catches "the app is completely broken" tier bugs.

1. **App loads.** Navigate to `APP_URL`. Verify no blank page, no 5xx, no pre-interaction console error.
2. **Zero console errors.** On the landing page, verify zero `error`-level console messages (warnings OK). This is the catch-all for "page loads but crashes on render" — missing imports, failed API calls, runtime exceptions.

**If the app is auth-gated** (you land on a sign-in screen), don't assume the login shape. Discover it: read the app's auth routes/components, or take an accessibility snapshot of the sign-in page and target the actual fields. Sign in with dev/test credentials, then run the baseline against the landed page. Note the auth flow you used so the check is reproducible.

## Step 2 — Build the session-derived plan

Read these signals in order:

1. `git diff $(git merge-base HEAD main)..HEAD --name-only` — which files changed on this branch
2. `git log $(git merge-base HEAD main)..HEAD --oneline` — what commits say about intent
3. Recent conversation context — what was agreed to build or fix this session

For each touched area, generate a targeted check. Generic examples — derive your own from the actual diff:

| Touched area | Add a check that |
| --- | --- |
| A route or page component | Navigates to that route, verifies it renders, no console errors |
| A shared UI component | Visits every surface that uses it, verifies each renders correctly |
| A form | Opens the form, fills required fields, submits, verifies expected outcome |
| An API or data layer | Loads a page that exercises that path, checks for network errors or 5xx |
| Navigation or layout | Clicks through affected nav items, verifies correct destination each time |
| A delete / destructive action | Opens the confirmation dialog, cancels (verifies dismiss), confirms (verifies removal) |

If nothing in the diff matches a known pattern, derive from the conversation: "we just added X — visit the page that exposes X and verify it renders."

### Components with scale-dependent behaviour

For components whose behaviour only triggers at volume — pagination, virtualization, bulk selection, infinite scroll — don't rely on low-data production routes to catch regressions. Maintain example routes seeded with enough data to exercise those states. A table with 2 rows never engages pagination; a table with 1000 rows will.

## Step 3 — Report

**Pass:** `Smoke test passed. Baseline (2) + N session checks, 0 errors. [screenshot]`

**Fail:** `Smoke test FAILED at [step description]: [exact what happened — console message verbatim, 5xx URL, or unexpected render]. [screenshot]. Blocking deploy.`

## Notes

- Console errors from browser extensions are ignored — only errors originating from your app's host count.
- If the dev server is serving stale content after a config change, kill and restart it before running.
- For routes behind feature flags or environment gates, document the gate inline in the per-route check rather than failing on missing UI.
