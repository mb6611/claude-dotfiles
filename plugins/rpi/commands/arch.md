---
description: Spec stage — interview the user and write a high-level .spec.md (no implementation)
argument-hint: "<feature idea or existing spec slug>"
---

You are in the **SPEC stage** of the spec → plan → implement cycle. This is where the user should spend most of their time. Your job is to help them think, not to rush to a document.

User input: $ARGUMENTS

## Hard rules

- Only create or edit files in `.claude/specs/`. Do **not** write or modify source code, configs, tests, or plan files. Do not scaffold anything.
- The spec says **what** and **why**, never **how**. Mention technology, file paths, APIs, or data schemas only when they are a hard constraint the user stated (e.g. "must use the existing Postgres", "must be a CLI flag").
- The spec is always the current proposal of what goes to the plan stage. Every line in it is something you believe should be built (or explicitly not built). Unknowns, open questions, and tradeoffs are raised **in chat**, never written into the file. Where something is undecided, the spec holds your recommended choice, and you tell the user in chat that it's an assumption to confirm.
- Keep it short: aim for under ~150 lines. If a section has nothing real to say, drop it.

## Writing principles

Every line must carry fundamental meaning. Work from first principles: what must be true for this feature to be correct? Write that, and nothing else.

- **No noise.** Cut filler, restatements, hedges, generic best practices ("should be performant", "handle errors gracefully"), and anything that would hold for any feature. If deleting a bullet loses nothing, delete it.
- **Say each thing once.** A fact lives in one item, and other places cite its ID. Don't echo requirements in goals, stories, or success criteria.
- **Requirements are baseline properties.** Each one states an invariant or observable behavior of the system: something that is always true, or always happens under a stated condition. Together they should be self-consistent, with no two items able to conflict. Anything else should follow from them.
- **Right granularity.** One item per independent property. Don't bundle unrelated behaviors into one item. Don't shatter one property into trivial sub-bullets either. If two items can't be true or false independently, they're one item.
- **Precise over complete.** A few exact properties beat many vague ones. Name the concrete condition and the concrete outcome.
- **Emphasis guides the skim.** Bold the one phrase per item that carries its point, so reading only the bold gives the gist. Italicize contrasts and caveats (*without fixing it*, *assumed*). ALL CAPS only for structural keywords: WHEN / IF / WHILE / SHALL in requirements, GIVEN / WHEN / THEN in acceptance criteria. Never emphasize a whole sentence, and leave an item plain when nothing in it stands out.

## Step 1: Orient

1. `ls .claude/specs .claude/plans 2>/dev/null` to see what exists. If `$ARGUMENTS` matches an existing spec slug, **resume** it (read it fully, re-scan it for gaps and continue the interview). If a plan already exists for it, warn that spec changes may invalidate the plan.
2. Pick a short kebab-case slug for a new feature (confirm it with the user in your first question round).
3. If this is an existing codebase, spawn 2–4 `Explore` subagents **in one message** so they run in parallel (thoroughness: medium), sized to how much of the codebase the idea touches. Give each a distinct focus so they don't overlap, for example: related existing features and how they behave; architecture boundaries and the areas the idea would touch; conventions and constraints (data model, auth, external integrations); prior art or half-built versions of the idea. Ask each for ≤300 words with file:line references. This informs your questions only; none of it goes in the spec unless it is a constraint. Skip for greenfield or trivial ideas.

## Step 2: Draft the skeleton early

Write `.claude/specs/<slug>.spec.md` right away. Use what you know, and fill the gaps with your best recommendation so the user can watch it take shape. In chat, list which parts are your assumptions. Template:

```markdown
# <Feature title>

## Problem
<Who has what problem, and why it matters now. Plain prose, no IDs.>

## Goals
- `G1` <Outcome, phrased from the user's point of view>

## Non-goals
- `NG1` <Explicitly out of scope — what we are NOT doing>

## User stories
### US1 (P1) <title>
<As a … I want … so that …>. Why P1: <reason>.
- `US1.1` GIVEN … WHEN … THEN …

## Requirements
- `R1` WHEN <trigger> THE SYSTEM SHALL <observable behavior, **key phrase bold**>.
- `R2` IF <unwanted condition> THEN THE SYSTEM SHALL <response>.

## Constraints & assumptions
- `C1` <Hard constraint from the user, or an assumption we're making>

## Key decisions
- `D1` <Decision> — because <why>. Rejected: <alternative> (<why not>).

## Edge cases
- `E1` <What happens when …>

## Success criteria
- `SC1` <Measurable, technology-agnostic outcome>
```

### IDs

Every item outside Problem starts with its ID in a code span (`` `R1` ``, never bold, so it stays a separate label from any bold phrase after it). The prefix names the type: `G` goal, `NG` non-goal, `US` user story (its acceptance criteria are `US1.1`, `US1.2`, …), `R` requirement, `C` constraint, `D` decision, `E` edge case, `SC` success criterion. The plan and reviews cite these IDs, so each item must make sense on its own when cited. Cite with the bare ID (`R4`, `R4–R7`, `US3`), never a section number.
- Add new items with the next free number for their prefix.
- Before a plan exists, you may renumber freely to keep things tidy. Once `.claude/plans/<slug>.plan.md` exists, **never renumber**. Delete an item by replacing its text with `*(removed)*` (keep the ID), and tell the user which plan references now need updating.

## Step 3: Interview loop (the main event)

Repeat until there are no significant open questions left **and** the user says they're happy:

1. Scan the spec for gaps across: scope boundaries, users & flows, data/entities (conceptually), error and edge cases, non-functional needs (performance, security, privacy, accessibility), integrations, constraints, terminology, vague adjectives ("fast", "simple", "secure"), and how we'll know it's done.
2. Ask the **highest-impact** questions first via `AskUserQuestion`, at most 4 per round. Make them multiple choice with your recommended option first (marked "(Recommended)") and a one-line reason, so the user can answer quickly. Free-text is always available to them.
3. Be a thinking partner, not a stenographer: challenge assumptions, point out tradeoffs and simpler alternatives, suggest things they may have missed, and push back on scope creep. Propose; let the user decide.
4. After each round, update the spec: fold answers into the right sections, replace or delete anything that no longer matches the agreed direction, and record notable choices under Key decisions. The file is the current proposal, not a history or a transcript.
5. Briefly tell the user what changed and what's still open. Ask if they want to go deeper anywhere before moving on.

## Step 4: Review

Once the first complete draft exists (no significant open questions left), invoke the `rpi:review` skill on the spec. After that, re-run it whenever later edits are significant (new or reworked requirements, scope changes, a direction change from the user). Skip it for small clarifications.

## Step 5: Approval

The spec is done when the user says so. Don't declare it finished on your own judgment. When they're happy, suggest `/rpi:plan <slug>`, ideally in a fresh session (`/clear`) so planning starts with clean context.
