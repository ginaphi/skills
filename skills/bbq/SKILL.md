---
name: bbq
description: Crystallise a planning, grilling, or design session into a Markdown record — every decision made, every path rejected, every open thread — written to docs/session/ as a plain .md file. Stack-agnostic; no special tooling required. Use after a grill/design/architecture discussion, or when the user says "/bbq", "crystallise this", "write up what we decided", or "session report".
---

# BBQ — Session Crystallisation Report

Turn a completed planning or design session into a durable Markdown record: every decision reached, every path rejected (as load-bearing as the chosen ones — they stop you re-litigating later), and every thread left open.

Output is a plain `.md` file in `docs/session/`. No special viewer or CLI is required — open it in whatever editor you use.

## Step 1 — Gather data

### From conversation context

Re-read the current session and extract:

**1. Decisions made** — each question that reached a conclusion (the user said "agree", "yes", or picked between options). For each capture: the question, the options on the table (including rejected ones), the chosen answer + one-sentence rationale, and whether a file change recorded it.

**2. Rejected paths** — branches explicitly closed ("no", "not worth it", "let's not"). Record them and why — they prevent the same debate next time.

**3. Open threads** — anything unresolved: "we'll figure that out later", a follow-up, a question that ran out of time.

### From file changes

```bash
# Resolve the base branch (main or master) and diff this branch against it
BASE=$(git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null)

git diff HEAD                                  # uncommitted (unstaged + working tree)
git diff --cached                              # staged
[ -n "$BASE" ] && git diff "$BASE"..HEAD       # committed on this branch
git status --short                             # new/untracked files
```

Look for changes to whatever documentation the project keeps — common spots: a glossary/`CONTEXT.md`, decision records (`docs/adr/`, `docs/decisions/`), specs/PRDs (`docs/prd/`), and agent rules/reference (`.claude/`). Adapt to what the repo actually has.

### Cross-reference — recorded vs unrecorded

A decision is **recorded** if a file change captures it (a new glossary term, a new decision record, an updated rule). It's **unrecorded** if it lives only in this conversation. Flag unrecorded decisions with `⚠ Unrecorded` — they're the risk.

## Step 2 — Write the Markdown report

```bash
DATE=$(date +%Y-%m-%d)
TIME=$(date +%H:%M)              # real clock time — never estimate
SLUG="<kebab-from-session-topic>"
mkdir -p docs/session
FILE="docs/session/${DATE}-bbq-${SLUG}.md"
```

Write `$FILE` using this shape (see [REFERENCE.md](REFERENCE.md) for the frontmatter contract + the Mermaid fork-diagram convention):

```markdown
---
date: YYYY-MM-DD
time: "HH:MM"
skill: bbq
context: <slug>
decisions: N
recorded: N
unrecorded: N
open_threads: N
---

# BBQ — <Session topic>

## Decisions made (N)

### Q1 — <Question label>

**Chosen:** <option> — <one-sentence rationale>
**Rejected:** <option A> (reason), <option B> (reason)
**Recorded in:** `<file>` / ⚠ unrecorded

---

### Q2 — <Question label>
...

## Doc changes

- ★ `<file>`: <what changed>

_(none)_

## Rejected paths

- ✗ <Branch label> — <why closed>

_(none)_

## Open threads

- → <Thread> — <why parked> → suggested next step

_(none — all threads resolved)_

---

N decisions crystallised · N recorded · N unrecorded · N open threads
```

When the decision tree is non-trivial, add a Mermaid `flowchart` code fence showing the resolved branches (chosen solid, rejected dotted) — see [REFERENCE.md](REFERENCE.md).

## Step 3 — Report to user

```
BBQ: docs/session/<filename>.md — N decisions (N recorded, N unrecorded), N open threads.
```

If any decisions are unrecorded, list their labels and recommend formalising them (a glossary term, a decision record, or a follow-up task) before closing the session.
