---
name: bump
description: Dependency-upgrade workflow for a JS/TS workspace (npm or pnpm). Surveys declared versions across the root + every workspace package.json, classifies each bump as at-latest / safe / major-needs-review / runtime-tied / CLI-managed, applies in logical buckets (one PR per bucket), verifies with the project's OWN scripts (typecheck/lint/build/test — whatever exists), and gives a revert order if something breaks. Use when the user says "/bump", "upgrade packages", "bump deps", "update dependencies", or asks whether a package is on the latest version.
---

# /bump — workspace dependency upgrade

A disciplined upgrade pass for a JS/TS monorepo or single package. Works with **npm** or **pnpm** workspaces. The golden rule: **bucket the work, verify with the project's own checks after each bucket, and never silent-bulk-bump.**

## Step 0 — Detect the setup

```bash
# package manager
ls pnpm-lock.yaml >/dev/null 2>&1 && PM=pnpm || PM=npm
# verify scripts the project actually defines (this is your gate — do not invent commands)
cat package.json | sed -n '/"scripts"/,/}/p'
# base branch
git merge-base HEAD main >/dev/null 2>&1 && BASE=main || BASE=master
git branch --show-current
```

A multi-package upgrade is a complex change — **branch first** (`git checkout -b deps-upgrade-<scope>`), don't commit straight to the base branch.

## Step 1 — Survey

```bash
npm outdated            # or: pnpm outdated -r   (recursive across workspaces)
```

`outdated` shows current / wanted / latest. Walk the root `package.json` plus every workspace `package.json`; skip workspace-internal links (versions like `"*"` or `workspace:*`). For each dep assign a category:

| Category               | Meaning                                   | Action                                                          |
| ---------------------- | ----------------------------------------- | --------------------------------------------------------------- |
| **at-latest**          | range already satisfies latest            | nothing to do                                                   |
| **safe**               | patch/minor within the same major         | ship in the sweep                                               |
| **major-needs-review** | major version jump                        | its own PR; read the changelog first                            |
| **runtime-tied**       | version-coupled to a framework/runtime    | bump as a coordinated set, pinned deliberately (see REFERENCE)  |
| **CLI-managed**        | a tool owns this dep (e.g. shadcn/ui)     | re-add via that tool's CLI; never hand-bump the range           |

## Step 2 — Propose buckets

One PR per bucket, stated explicitly (bumping / skipping / why — no silent bulk-bumps):

1. **Safe sweep** — all `safe` deps together.
2. **Majors** — one PR per major or per related cluster (e.g. a framework + its `react`/`react-dom` + matching `@types/*` move together).
3. **Runtime-tied** — pin to the runtime. The classic trap: `@types/node` pinned to the **installed Node major** (`node --version`) — bumping types ahead of the runtime makes the editor suggest APIs that crash. An ORM's client + CLI + adapter move as one set.
4. **CLI-managed** — re-add via the owning CLI, don't touch the range.

## Step 3 — Changelog scan (before applying)

Scan changelogs for every `major-needs-review` dep and any `safe` bump in a volatile family (frameworks, ORMs, auth, RPC layers — these ship breaking changes disguised as minors). Use the project's doc-lookup tool (Context7 / the package's GitHub releases).

**Park a finding** only when it (1) replaces a pattern you hand-rolled, (2) contradicts a documented convention/decision in the repo, or (3) needs migration beyond install (renamed export, changed config, codemod). Everything else → one-line note in the summary. Record parked items as short notes (e.g. under `docs/session/` or `docs/backlog/`) and list them as **"Worth grilling"** — don't block the run.

## Step 4 — Apply

```bash
# npm — target a workspace by its package "name", or root with no flag
npm install <pkg>@<ver> -w <workspace-name>
npm install <pkg>@<ver>                       # root dep/devDep

# pnpm — --filter by package name, or -w for the workspace root
pnpm --filter <workspace-name> add <pkg>@<ver>
pnpm add -w <pkg>@<ver>                        # root
```

Combine multiple deps for one workspace in a single command. Never hand-edit the lockfile — let the package manager regenerate it. For **CLI-managed** deps, re-run that tool (e.g. `npx shadcn@latest add <component> --overwrite`).

## Step 5 — Verify (project's own scripts)

After **each bucket**, run whatever checks the project defines — read them from Step 0, don't assume. Typical:

```bash
<pm> run lint          # or: npx biome check . / eslint .
<pm> run typecheck     # or check-types / tsc --noEmit
<pm> run build
<pm> test              # ONLY if a test script exists — never invent one
```

If the project has no test runner, say so and rely on typecheck + build. A build that runs codegen (e.g. an ORM client generate step) will catch client drift — if it trips right after an ORM bump, regenerate the client and rebuild before blaming the dep.

## Step 6 — Bisect on regression

Don't blame deps before verifying. Most "the upgrade broke X" is unrelated drift the upgrade exposed (stale generated code, a renamed import, an old fixture). Revert in blast-radius order — **toolchain → framework → auth → data layer → leaf libs** — re-running verify after each, until the offender is isolated.

## Step 7 — Commit

One PR per bucket, conventional commits, each body listing **what was skipped and why**:

- `chore(deps): sweep patches + minors (N deps)`
- `chore(deps): upgrade <pkg> vX → vY`
- `chore(deps): pin @types/node to runtime <N>`

## Output

Report to the user: a table of what bumped (package · workspace · from → to · category), a list of what was skipped + why, per-command verify status, PR link(s), and a **"Worth grilling"** line per parked changelog finding (omit if none).

See [REFERENCE.md](REFERENCE.md) for the families that are usually runtime-tied or CLI-managed.
