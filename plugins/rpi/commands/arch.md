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
- **Say each thing once.** A fact lives in one item, and other places cite its § number. Don't echo requirements in goals, stories, or success criteria.
- **Requirements are baseline properties.** Each one states an invariant or observable behavior of the system: something that is always true, or always happens under a stated condition. Together they should be self-consistent, with no two items able to conflict. Anything else should follow from them.
- **Right granularity.** One item per independent property. Don't bundle unrelated behaviors into one item. Don't shatter one property into trivial sub-bullets either. If two items can't be true or false independently, they're one item.
- **Precise over complete.** A few exact properties beat many vague ones. Name the concrete condition and the concrete outcome.

## Step 1: Orient

1. `ls .claude/specs .claude/plans 2>/dev/null` to see what exists. If `$ARGUMENTS` matches an existing spec slug, **resume** it (read it fully, re-scan it for gaps and continue the interview). If a plan already exists for it, warn that spec changes may invalidate the plan.
2. Pick a short kebab-case slug for a new feature (confirm it with the user in your first question round).
3. If this is an existing codebase, spawn 2–4 `Explore` subagents **in one message** so they run in parallel (thoroughness: medium), sized to how much of the codebase the idea touches. Give each a distinct focus so they don't overlap, for example: related existing features and how they behave; architecture boundaries and the areas the idea would touch; conventions and constraints (data model, auth, external integrations); prior art or half-built versions of the idea. Ask each for ≤300 words with file:line references. This informs your questions only; none of it goes in the spec unless it is a constraint. Skip for greenfield or trivial ideas.

## Step 2: Draft the skeleton early

Write `.claude/specs/<slug>.spec.md` right away. Use what you know, and fill the gaps with your best recommendation so the user can watch it take shape. In chat, list which parts are your assumptions. Template:

```markdown
# <Feature title>

## §1 Problem
§1.1 <Who has what problem, and why it matters now.>

## §2 Goals
- §2.1 <Outcome, phrased from the user's point of view>

## §3 Non-goals
- §3.1 <Explicitly out of scope — what we are NOT doing>

## §4 User stories
### §4.1 (P1) <title>
<As a … I want … so that …>. Why P1: <reason>.
- §4.1.1 Given … When … Then …

## §5 Requirements
- §5.1 WHEN <trigger> THE SYSTEM SHALL <observable behavior>.
- §5.2 IF <unwanted condition> THEN THE SYSTEM SHALL <response>.

## §6 Constraints & assumptions
- §6.1 <Hard constraint from the user, or an assumption we're making>

## §7 Key decisions
- §7.1 <Decision> — because <why>. Rejected: <alternative> (<why not>).

## §8 Edge cases
- §8.1 <What happens when …>

## §9 Success criteria
- §9.1 <Measurable, technology-agnostic outcome>
```

### Numbering

Every section is `§N` and every item in it is `§N.M` (nested items `§N.M.K`), so the plan and reviews can cite exact parts. Each numbered item must make sense on its own when cited.
- Keep the section numbers above even if a section is dropped. Don't shift later sections up.
- Add new items at the end of their section with the next free number.
- Before a plan exists, you may renumber freely to keep things tidy. Once `.claude/plans/<slug>.plan.md` exists, **never renumber**. Delete an item by replacing its text with `*(removed)*`, and tell the user which plan references now need updating.

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
