---
description: Plan stage — research the codebase and turn an approved spec into detailed .plan.md file(s)
argument-hint: "<spec slug or path>"
---

You are in the **PLAN stage** of the spec → plan → implement cycle. Turn an approved spec into instructions detailed enough that a fresh agent could implement each part without asking questions.

User input: $ARGUMENTS

## Hard rules

- Only create or edit files in `.claude/plans/` (and the spec, if the user agrees it needs changes). Do **not** write or modify source code.
- The final plan has **no open questions**. Resolve them with the user or by reading code.
- No noise. Every task is a concrete change that someone must make, with no restated context, generic advice, or tasks that exist only to look thorough. Size tasks so each is one meaningful change: don't bundle unrelated edits, and don't split one edit into trivial steps. Say each thing once and cite it by ID.
- Every task cites the spec items it serves (`R2`), or is explicitly justified. Nothing the spec lists as a non-goal gets planned.

## Step 1: Load the spec

1. Find the spec: `.claude/specs/<slug>.spec.md` (if `$ARGUMENTS` is empty, list specs that have no plan yet, and ask which).
2. Read it fully. Running `/rpi:plan` means the user considers the spec done. If you see it still has obvious holes, say so briefly and offer to go back to `/rpi:arch`.
3. If a plan already exists, read it and resume rather than starting over.

## Step 2: Research (parallel, read-only)

Spawn 2–4 `Explore` subagents **in one message**, sized to the spec (1 for a tiny feature). Give each a distinct focus, for example:
- Where the code touched by the spec's requirements lives today, and how it flows
- Existing patterns/conventions to mirror (similar features, error handling, tests)
- Build, test, lint, typecheck, and migration commands; CI config
- Integration points and anything risky (shared state, public APIs, data migrations)

Ask each for ≤400 words with `file:line` references, no file dumps. Then read the few most critical files yourself to verify their claims before relying on them.

## Step 3: Agree on the outline first

Before writing details, present a short outline to the user:
- Approach in a few sentences, mapped to the spec's requirements
- Key technical decisions with tradeoffs (use `AskUserQuestion`, recommended option first, for anything the code doesn't answer)
- Proposed units of work with dependencies, and whether to split into subplans (see below)

Iterate until the user agrees. If research shows the spec is wrong or incomplete, say so and, with the user's agreement, update the spec (recording the change under Key decisions) before planning around it.

## Step 4: Write the plan

### Single plan vs master + subplans

Use a single `.claude/plans/<slug>.plan.md` by default. Split into a **master plan** plus **subplans** when any of these hold: 3+ independent workstreams, more than ~25 tasks, or the plan would exceed ~300 lines.

- Master: `.claude/plans/<slug>.plan.md` — overview, decisions, waves, coverage; links to subplans instead of listing their tasks.
- Subplans: `.claude/plans/<slug>/NN-<unit>.plan.md` — one per unit of work, self-contained enough that a subagent can execute it with only that file plus the spec items it cites.

Draw unit boundaries along **file ownership**: two units that can run in parallel must not edit the same files. Shared foundations (types, schema, config) go in an earlier unit that others depend on.

### Plan template (master or single)

```markdown
# <Feature> — Implementation Plan

**Spec:** [`.claude/specs/<slug>.spec.md`](../specs/<slug>.spec.md)

## Overview
<What we're building and the approach, 3–6 sentences.>

## Current state
- <What exists today, with file:line references>

## Key decisions
- <Decision> — <why>. Rejected: <alternative>. (D2)

## What we're NOT doing
- <Implementation-level exclusion> (NG1)

## Units
### U1 <unit name>
- **Depends on**: —
- **Owns**: `src/foo/*.ts`, `src/foo.test.ts`
- [ ] **U1.1** Add `parseThing(input: Raw): Thing` in `src/foo/bar.ts`. Validate X, return Y on failure. (R1, E2)
- [ ] **U1.2** Tests for `parseThing` happy path and the E2 case in `src/foo.test.ts`. (R1)
- [ ] **U1.3** verify: `npm test -- foo` passes
- [ ] **U1.4** verify: `npm run typecheck` passes
- [ ] **U1.5** manual: <what a human should check> (SC1)

## Execution waves
| Wave | Units | Notes |
|------|-------|-------|
| 1 | U1 | foundation |
| 2 | U2, U3 | parallel, disjoint files |

## Testing strategy
- <How the spec's success criteria are verified end to end>

## Risks & rollback
- <Risk> — <mitigation>

## Coverage
| Spec | Plan |
|------|------|
| R1 | U1.1, U1.2 |
| SC1 | U1.5 |
```

### IDs and references

- Units are `U1`, `U2`, … and their tasks `U1.1`, `U1.2`, …. Other plan sections are plain bullets with no IDs. Once implementation starts, never renumber; mark dropped tasks `*(removed)*` and keep the ID.
- Cite spec items by their bare IDs (`R5`, `E2`, `SC1`); the prefixes never collide with plan IDs. Every task cites the spec items it serves. Every spec goal, user-story acceptance criterion, requirement, edge case and success criterion appears in the coverage table, or the plan explains why it's not covered.
- Each task ends in a checkbox and is phrased so an agent with no context can do it.

### Subplans

A subplan `.claude/plans/<slug>/NN-<unit>.plan.md` holds the full content of master unit `UN` (`NN` is `N` zero-padded, so `U2` lives in `02-<unit>.plan.md`) and keeps that numbering (its tasks are `UN.1`, `UN.2`, …). It starts with:

```markdown
# UN <unit name>

**Spec:** [`.claude/specs/<slug>.spec.md`](../../specs/<slug>.spec.md) · **Master plan:** [`<slug>.plan.md`](../<slug>.plan.md)

- **Depends on**: U1
- **Owns**: `src/api/**`
```

In the master, that unit is one line: `### UN <name> → [subplan](<slug>/NN-<unit>.plan.md)` with its Depends on / Owns.

### Level of detail

Closer to implementation than the spec: exact file paths, functions/types to add or change, signatures, data shapes, migration steps, and the order of changes. Include short code snippets only where precision matters (a tricky signature, a SQL migration). Don't write the implementation itself.

## Step 5: Review and approval

1. Invoke the `rpi:review` skill on the plan (it covers subplans too). Re-run it after significant later edits, such as reworked units, dependencies, or ownership, but not for small fixes.
2. Walk the user through the plan briefly, focusing on decisions and anything surprising.
3. The plan is done when the user says so. When they're happy, suggest `/rpi:implement <slug>`, ideally after `/clear`.
