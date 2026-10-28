# Past cases

Real moments from this user's conversations, with where each lesson belongs. They show the judgment, not wording to copy. Add new cases at the bottom, one line each: what happened → verdict.

## Saved, and where each belongs
- "use pnpm not npm btw" mid-scaffold → user-wide preference → `plugins/memory/CLAUDE.md`. (It first landed in one project's auto-memory, which was the wrong scope.)
- "please stop using phrases like load-bearing, smoke testing… you don't speak normally" → user-wide style → `plugins/memory/CLAUDE.md`. (Also first landed in project memory.)
- Spec looked unreadable with `§` on every line → shown on the real spec first, then "okay let's do it" → rpi command templates.
- "the number blends in with the first bolded statement" → previewed on the spec, then approved → rpi templates (IDs as code spans).
- "/git:commit and push", typed by hand twice in a few hours → new command `/git:save`.
- "is there a way to set a user level memory from a plugin" (user-level CLAUDE.md wouldn't carry to a new machine) → the `memory` plugin itself.
- A run of corrections while building rpi ("spawn multiple explore agents", "raise unknowns in chat", "LLMs like to nitpick") → each folded into the rpi command it governs.
- A long explanation of "idea savings" and physics-style rigor → the `/learn:idea-savings` command, where it's used. A separate memory of the same thing was a duplicate.

## Removed, which was the right call
- Hook scripts, a stage script, a `--quick` flag, a status command in rpi → "having stage as a script is dumb, the agent should just know" → deleted, not patched.
- A tmux rule Claude added → "is there a claude native way" → reverted, native option used.
- plugin-dev plugin → "unnecessary now that models are really good" → removed.

## Not worth saving
- Personal one-off questions (leases, jury duty, a movie ending).
- One-off fixes (a neovim treesitter error, a notification permission).
- Explanations the user asked for to understand something (REA internals, another project's architecture).
- Task-level choices ("last 4 years", "do we really need to stagger it").

## Still open
- "Don't do anything yet, just show me" said 5+ times across sessions → candidate for `plugins/memory/CLAUDE.md`, not yet confirmed.
- "use cdp / start arc in debugging mode" for browser work, said in 3 sessions → candidate user-wide note, not yet confirmed.
