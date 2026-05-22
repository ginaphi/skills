---
name: thermo
description: Structural code quality audit — code judo, 1k-line rule, spaghetti detection, abstraction boundary enforcement. Reviews branch vs main diff. Use when you want a harsh structural review before opening a PR, or when the user says "/thermo", "thermo review", "thermonuclear review", or "code quality audit".
---

Goal: Deliver a structural code quality verdict on every change in this branch vs `main`.
Success means: Every meaningful structural regression, missed simplification, and boundary leak is surfaced with priority and a concrete remediation path. Low-value cosmetic nits are suppressed when structural issues dominate.
Stop when: All changed files are evaluated against the rubric and findings are printed to console.

## Step 1 — Gather inputs (run in parallel)

- `git diff main...HEAD` — full line-level diff of this branch
- `git diff main...HEAD --name-only` — list of changed files

Then read the **full content** of every changed file (not just diff hunks) — cross-file impact is only visible from the whole module.

If there is no diff vs `main` (clean branch or already merged), report that and stop.

## Step 2 — Apply the structural rubric

### Core intent

Be **ambitious** about code structure. Do not merely flag local cleanup. Actively hunt for "code judo" moves: restructurings that preserve behaviour while making the implementation dramatically simpler, smaller, and more direct. Prefer the solution that makes the code feel inevitable in hindsight.

---

### Non-negotiable standards

**0. Structural simplification first.**
Look for opportunities to reframe a change so that whole branches, helpers, modes, conditionals, or layers disappear entirely. If you see a path to delete complexity rather than rearrange it, push hard for that path. Do not stop at "this could be a bit cleaner."

**1. The 1k-line rule.**
A PR must not push a file from under 1k lines to over 1k lines without a compelling structural reason. Treat this as a strong quality smell by default. Prefer extracting helpers, subcomponents, or local abstractions. If the diff crosses that threshold, explicitly say so and ask whether the code should be decomposed first.

**2. No spaghetti growth.**
Be highly suspicious of new ad-hoc conditionals, scattered special cases, or one-off branches inserted into unrelated flows. If a change adds "weird if statements in random places", treat that as a design problem, not a stylistic nit. Push the logic into a dedicated abstraction, helper, state machine, or separate module instead.

**3. Clean the design, not just the behaviour.**
If behaviour stays the same while the structure could become meaningfully cleaner, push for the cleaner version. Do not rubber-stamp "it works" implementations that leave the codebase messier. Strongly prefer simplifications that remove moving pieces altogether over refactors that merely spread the same complexity.

**4. Direct, boring, maintainable code over hacky or magical code.**
Treat brittle, ad-hoc, or "magic" behaviour as a quality problem. Be sceptical of generic mechanisms that hide simple data-shape assumptions. Flag thin abstractions, identity wrappers, or pass-through helpers that add indirection without buying clarity.

**5. Type and boundary cleanliness.**
Question unnecessary optionality, `unknown`, `any`, or cast-heavy code when a clearer type boundary could exist. Prefer explicit typed models over loosely-shaped ad-hoc objects. If a branch relies on silent fallback to paper over an unclear invariant, ask whether the boundary should be made explicit instead.

**6. Canonical layer discipline.**
Call out feature logic leaking into shared paths or implementation details leaking through APIs. Prefer existing canonical utilities and helpers over bespoke one-offs. Push code toward the right package, service, or module instead of normalising architectural drift.

**7. Orchestration and atomicity.**
If independent work is serialised for no good reason, ask whether the flow should be simpler. If related updates can leave state half-applied, push for a more atomic structure. Do not over-index on micro-optimisations, but do flag avoidable orchestration complexity.

---

### Primary review questions

For every meaningful change:

- Is there a "code judo" move that would make this dramatically simpler?
- Can this be reframed so fewer concepts, branches, or helper layers are needed?
- Did the diff add branching complexity where a better abstraction should exist?
- Is this logic living in the right file and layer?
- Did this change push a file past a healthy size boundary?
- Are there repeated conditionals that signal a missing model or missing helper?
- Is this abstraction actually earning its keep, or is it just a wrapper?
- Did the diff introduce casts, optionality, or ad-hoc object shapes that obscure the real invariant?
- Is this orchestration more sequential or less atomic than it needs to be?

---

### Preferred remedies

When you identify a structural problem, prefer:

- Delete a whole layer of indirection rather than polishing it
- Reframe the state model so conditionals disappear instead of getting centralised
- Extract a helper or pure function
- Split a large file into smaller focused modules
- Move feature-specific logic behind a dedicated abstraction
- Replace condition chains with a typed model or explicit dispatcher
- Separate orchestration from business logic
- Delete wrappers that do not meaningfully clarify the API
- Reuse the existing canonical helper instead of introducing a near-duplicate

Do not be satisfied with "maybe rename this" feedback when the real issue is structural.
Do not be satisfied with a merely cleaner version of the same messy idea if there is a plausible path to a much simpler idea.

---

## Output format

Print findings in this priority order:

1. Structural code-quality regressions
2. Missed code-judo opportunities — dramatic simplifications that are visible but untaken
3. Spaghetti / branching complexity increases
4. Boundary / abstraction / type-contract problems
5. File-size and decomposition concerns
6. Modularity and abstraction issues
7. Legibility and maintainability concerns

For each finding:
- Lead with the file and the nature of the problem (one direct sentence)
- Explain why it's a problem (one to two sentences)
- Propose the preferred remedy

Suppress cosmetic nits when structural issues exist. Prefer a small number of high-conviction findings over a long list of minor observations.

If no structural issues are found, say so plainly. The approval bar is:

- No clear structural regression
- No obvious missed simplification when a code-judo path is visible
- No unjustified file-size explosion past 1k lines
- No obvious spaghetti-growth from special-case branching
- No hacky or magical abstraction that makes the code harder to reason about
- No unnecessary wrapper/cast/optionality that obscures the real design
- No architecture-boundary leak or avoidable canonical-helper duplication

If the bar is met, say: **Structural review passed.** followed by a one-sentence summary of what was reviewed.
If the bar is not met, lead with the most severe finding and work down.

---

## Tone

Be direct, serious, and demanding about quality.
Do not be rude, but do not soften major maintainability issues into mild suggestions.
If the code is making the codebase messier, say so clearly.

Good phrases to use when warranted:

- `this pushes the file past 1k lines — can we decompose this first?`
- `this adds another special-case branch into an already busy flow — can we move this behind its own abstraction?`
- `this works, but it makes the surrounding code more spaghetti — let's keep the behaviour and restructure the implementation`
- `this feels like feature logic leaking into a shared path — can we isolate it?`
- `this abstraction isn't earning its keep — can we just keep the direct flow?`
- `why does this need a cast / optional here? can we make the boundary more explicit instead?`
- `this looks like a bespoke helper for something we already have elsewhere — can we reuse the canonical one?`
- `i think there's a code-judo move here that makes this much simpler — can we reframe this so these branches disappear?`
- `this refactor moves complexity around, but doesn't really delete it — is there a way to make the model itself simpler?`
