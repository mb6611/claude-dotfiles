---
description: Plan stage — research the codebase and turn an approved spec into detailed .plan.md file(s)
argument-hint: "<spec slug or path>"
---

You are in the **PLAN stage** of the spec → plan → implement cycle. Turn an approved spec into instructions detailed enough that a fresh agent could implement each part without asking questions.

User input: $ARGUMENTS

## Hard rules

- Only create or edit files in `.claude/plans/` (and the spec, if the user agrees it needs changes). Do **not** write or modify source code.
- The final plan has **no open questions**. Resolve them with the user or by reading code.
- No noise. Every task is a concrete change that someone must make, with no restated context, generic advice, or tasks that exist only to look thorough. Size tasks so each is one meaningful change: don't bundle unrelated edits, and don't split one edit into trivial steps. Say each thing once and cite it by §.
- Every task cites the spec items it serves (`spec §5.2`), or is explicitly justified. Nothing the spec lists as a non-goal gets planned.

## Step 1: Load the spec

1. Find the spec: `.claude/specs/<slug>.spec.md` (if `$ARGUMENTS` is empty, list specs that have no plan yet, and ask which).
2. Read it fully. Running `/rpi:plan` means the user considers the spec done. If you see it still has obvious holes, say so briefly and offer to go back to `/rpi:arch`.
3. If a plan already exists, read it and resume rather than starting over.

## Step 2: Research (parallel, read-only)

Spawn 2–4 `Explore` subagents **in one message**, sized to the spec (1 for a tiny feature). Give each a distinct focus, for example:
- Where the code touched by the spec's requirements (spec §5) lives today, and how it flows
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

## §1 Overview
§1.1 <What we're building and the approach, 3–6 sentences.>

## §2 Current state
- §2.1 <What exists today, with file:line references>

## §3 Key decisions
- §3.1 <Decision> — <why>. Rejected: <alternative>. (spec §7.2)

## §4 What we're NOT doing
- §4.1 <Implementation-level exclusion> (spec §3.1)

## §5 Units
### §5.1 <unit name>
- **Depends on**: —
- **Owns**: `src/foo/*.ts`, `src/foo.test.ts`
- [ ] §5.1.1 Add `parseThing(input: Raw): Thing` in `src/foo/bar.ts`. Validate X, return Y on failure. (spec §5.1, §8.2)
- [ ] §5.1.2 Tests for `parseThing` happy path and §8.2 edge case in `src/foo.test.ts`. (spec §5.1)
- [ ] §5.1.3 verify: `npm test -- foo` passes
- [ ] §5.1.4 verify: `npm run typecheck` passes
- [ ] §5.1.5 manual: <what a human should check> (spec §9.1)

## §6 Execution waves
| Wave | Units | Notes |
|------|-------|-------|
| 1 | §5.1 | foundation |
| 2 | §5.2, §5.3 | parallel, disjoint files |

## §7 Testing strategy
- §7.1 <How the spec's success criteria (spec §9) are verified end to end>

## §8 Risks & rollback
- §8.1 <Risk> — <mitigation>

## §9 Coverage
| Spec | Plan |
|------|------|
| §5.1 | §5.1.1, §5.1.2 |
| §9.1 | §5.1.5 |
```

### Numbering and references

- Same rules as the spec. Every section is `§N`, every item `§N.M`, and tasks are `§5.U.T`. Once implementation starts, never renumber; mark dropped items `*(removed)*`.
- A bare `§` points inside the plan. Spec references are always written `spec §N.M`. Every task cites the spec items it serves. Every spec item in §2, §4, §5, §8 and §9 appears in the coverage table, or the plan explains why it's not covered.
- Each task ends in a checkbox and is phrased so an agent with no context can do it.

### Subplans

A subplan `.claude/plans/<slug>/NN-<unit>.plan.md` holds the full content of master unit `§5.N` (`NN` is `N` zero-padded, so `§5.2` lives in `02-<unit>.plan.md`) and keeps that numbering (its tasks are `§5.N.1`, `§5.N.2`, …). It starts with:

```markdown
# §5.N <unit name>

**Spec:** [`.claude/specs/<slug>.spec.md`](../../specs/<slug>.spec.md) · **Master plan:** [`<slug>.plan.md`](../<slug>.plan.md)

- **Depends on**: §5.1
- **Owns**: `src/api/**`
```

In the master, that unit is one line: `### §5.N <name> → [subplan](<slug>/NN-<unit>.plan.md)` with its Depends on / Owns.

### Level of detail

Closer to implementation than the spec: exact file paths, functions/types to add or change, signatures, data shapes, migration steps, and the order of changes. Include short code snippets only where precision matters (a tricky signature, a SQL migration). Don't write the implementation itself.

## Step 5: Review and approval

1. Invoke the `rpi:review` skill on the plan (it covers subplans too). Re-run it after significant later edits, such as reworked units, dependencies, or ownership, but not for small fixes.
2. Walk the user through the plan briefly, focusing on decisions and anything surprising.
3. The plan is done when the user says so. When they're happy, suggest `/rpi:implement <slug>`, ideally after `/clear`.
