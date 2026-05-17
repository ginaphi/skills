---
name: genui
description: Primary trigger — /genui command or "genui this". Sketches a throwaway `.openui` UI prototype in `docs/prototype/` for live preview in the Genui VS Code extension. Also triggers on: "sketch a [page]", "mock up [thing]", "let me see a few options", "prototype this design", "draft a UI for X". Not for business-logic exploration or terminal apps.
---

# Genui — sketch UIs in conversation

Generate `.openui` UI prototype files in `docs/prototype/` for the developer to render live in the Genui VS Code extension. **Throwaway by design** — the answer to "what should this look like" is what matters, not the sketch.

For **how to author OpenUI Lang syntax**, read the bundled [openui-lang](./openui-lang/SKILL.md) reference. This skill is about *when/where/why* — not syntax.

## When to invoke

**Primary:** `/genui` command, or the user says "genui this".

**Also triggers on:**
- "sketch a [page / screen / layout]"
- "mock up a [thing]"
- "let me see a few options for X"
- "draft a UI for X"
- "prototype this design"
- "what should this look like"

**Do not invoke** for: state-machine sanity checks, business-logic exploration, terminal apps.

## The 6-step workflow

**1. State the question.** Write it as the file header (see File-header convention). If the question isn't crisp, ask first — a vague question produces a vague sketch.

**2. Single design or variants?** Default: one file. Variants (2–3, max 3) only when the user explicitly wants to compare. Hold variants to structurally different — different layout, hierarchy, primary affordance. Colour tweaks are not variants.

**3. Pick location and filename.** Folder: `docs/prototype/` at workspace root (create if missing). Kebab-case slug: `settings-page.openui`. Variants: `<slug>-a`, `-b`, `-c`. Re-prototyping the same area? Version it: `settings-page-v2`. Never overwrite silently.

**4. Generate.** Read `openui-lang/SKILL.md` then validate:
```bash
node .claude/skills/genui/openui-lang/scripts/validate.mjs docs/prototype/<slug>.openui
```
The validator catches parse-level errors only. **Component prop-schema errors surface as a red banner in the Genui preview.** Treat that banner as ground truth.

**5. Hand off.** Tell the user the file path and instruct: open from Explorer → run **Genui: Open Preview** (`Cmd+Shift+P → Genui`). For variants, list all paths — each file gets its own preview tab. If a red error banner appears, paste the error and fix the file; the watcher re-renders on save.

**6. Clean up.** A prototype that lingers is a prototype that lies. Delete the file when done. Only write to `docs/prototype/NOTES.md` (5–15 lines: question, outcome, why) if the exploration produced learning worth keeping — then delete the prototype files.

## File-header convention

Every `.openui` starts with three comment lines: `# PROTOTYPE — <question>`, `# Generated: <YYYY-MM-DD>`, `# Status: open | answered | abandoned`. Status lifecycle: `open` → `answered` or `abandoned` → deleted.

## Anti-patterns

- **Don't put `.openui` files in `src/`, `app/`, or the source tree.** Sketches aren't code. `docs/prototype/` is where they belong.
- **Don't generate 5+ variants.** 2–3 is the sweet spot. Beyond that, variants stop being radically different.
- **Don't make colour-only variants.** "Same layout, different palette" is a tweak, not a prototype.
- **Don't leave prototype files lingering across sessions.** If the design was answered, delete. If not, status it as `abandoned` or `open` and revisit explicitly.
- **Don't reuse a slug silently.** Re-prototyping the same area? Version it (`-v2`). Old prototypes stay readable in git history.
- **Don't promote the `.openui` directly to production.** The renderer is a preview tool. To ship the design, write the real React (or other framework) component the prototype suggested.

## What this skill is NOT

- Not the syntax reference — that's the bundled [openui-lang](./openui-lang/SKILL.md).
- Not a code generator that turns a prototype into React. The developer or another agent does that translation.
- Not a UI library — `@openuidev/react-ui` is. We just render its components from text.
- Not for production UI. Always throwaway.

## See also

- [openui-lang](./openui-lang/SKILL.md) — how to write `.openui` files correctly. Required reading before generating any `.openui` content.
- The [Genui VS Code extension](https://github.com/ginaphi/genui) — open any `.openui` file, run "Genui: Open Preview" to render. Live re-renders on save.
