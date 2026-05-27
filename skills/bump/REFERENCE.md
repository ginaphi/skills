# Bump Reference — families that need care

Generic patterns for classifying a dependency. These are *shapes* to recognise, not a fixed package list — apply them to whatever the project actually depends on.

## Runtime-tied sets (bump as a coordinated group, never one-at-a-time)

- **UI framework + its peers + types.** A frontend framework moves together with its React/Vue/etc. runtime and the matching `@types/*`. Example shape: `react` + `react-dom` + `@types/react` + `@types/react-dom` are a quartet — bump all four or none.
- **Meta-framework majors.** Next.js / Nuxt / SvelteKit / Remix majors are their own PR — they often require config changes and codemods.
- **`@types/node` ↔ Node runtime.** Pin `@types/node` to the **installed Node major** (`node --version`). Types ahead of the runtime make the editor autocomplete APIs that throw at runtime.
- **ORM client + CLI + adapter.** An ORM's client, its CLI/migrate tool, and any DB adapter must share a major (e.g. a `*/client` + the `prisma`-style CLI + a `*-adapter-*`). A client bump usually needs a regenerate + rebuild.
- **RPC / auth / validation layers.** RPC libraries, auth libraries, and schema/validation libs ship breaking changes inside minors — scan their changelog even for `safe`-looking bumps.

## CLI-managed deps (re-add via the tool, don't hand-bump the range)

Some deps are owned by a generator/CLI, and editing their version range by hand gets reverted on the next regenerate:

- **shadcn/ui** primitives — re-add with `npx shadcn@latest add <component> --overwrite`; bump the underlying Radix/Base-UI packages together via the CLI, not manually.
- Any codegen output or scaffolded component set with its own "add/update" command.

## Cleanup signals (not a bump)

- **Two libraries doing the same job** (e.g. two icon sets, two date libs, two state managers) usually means a half-finished migration — flag for consolidation rather than bumping both.
- **A dep with zero importers** — surfaced by the build/lint or a dead-code check — should be removed, not upgraded.

## The discipline

- Bucket by risk; one PR per bucket.
- Verify with the **project's own** lint/typecheck/build/test scripts after every bucket.
- Read changelogs for majors and volatile families before applying.
- On regression, revert in blast-radius order (toolchain → framework → auth → data → leaf) and re-verify between steps.
