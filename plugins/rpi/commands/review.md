---
description: Review a .spec.md or .plan.md with fresh eyes — auto-fix small issues, raise only big ones to the user
argument-hint: "[spec/plan path or slug]"
---

Review a spec or plan with fresh eyes, before the user has to find the problems.

Target: $ARGUMENTS

Resolve the target:
- A `.spec.md` path → **spec review**.
- A `.plan.md` path → **plan review** of the master plan plus all its subplans, checked against its spec.
- A slug → review the latest stage that exists for it: the plan if there is one, otherwise the spec.
- Empty → the most recently modified file in `.claude/specs/` or `.claude/plans/`.

## Step 1: Independent review (subagent)

Spawn **one** subagent (`subagent_type: "Plan"`, read-only) so the review isn't biased by this conversation. For a plan with more than ~4 subplans, spawn one per group of subplans plus one for the master, all in a single message. Give it the brief below, with only the checklist that matches the target.

```
Review <path(s)>. <For a plan: Check it against its spec at <spec path>.> Read everything fully.

Look only for issues that would cause a wrong build, rework, a stalled next stage, or a failed
parallel run.

<CHECKLIST>

Common to both:
- Noise: bullets that are filler, generic, restated elsewhere, or would hold for any feature.
  Items bundled together that should be separate, or split up when they're really one thing.
  Fix by deleting or merging.
- IDs: items missing an ID, duplicate or wrong-prefix IDs, items that don't make sense when cited
  alone.
- TODOs, placeholders, or open questions written into the file. It should read as a settled
  proposal, and questions belong in chat. Raise the underlying question as an ASK and remove it.
- <If the user dismissed findings earlier in this conversation: "Already dismissed, do not re-raise: …">

Be objective. LLM reviewers tend to nitpick, so test every finding before reporting it: "If this
were left as is, what concretely goes wrong in the next stage or the build?" If you can't name a
concrete consequence, it isn't a finding. Don't report style preferences, wording you'd merely phrase
differently, "could also consider" ideas, or alternatives that are just different. A file with no real
problems should get an empty list. That's a good outcome, not a failed review.

Severity is the consequence if the issue is left in:
- CRITICAL: the build would be wrong or unusable. A core requirement is missing or contradicted, the
  plan implements the wrong thing, or parallel units would clobber each other.
- HIGH: likely rework or a stalled stage. Ambiguity an implementer would have to guess at, an
  untestable requirement, a coverage gap, a wrong assumption about the code.
- MEDIUM: a real but contained problem. A missing edge case with an obvious answer, an inconsistency
  between items, a broken ID reference, an unrunnable verify step.
- LOW: costs clarity, not correctness. Noise, duplication, ID slips, misplaced detail.

Disposition is whether a human is needed, independent of severity:
- ASK: genuinely ambiguous. Reasonable people could choose differently and the files and code don't
  settle it (product behavior, priority, scope, a tradeoff, intent). Give 2–3 options and your
  recommendation. LOW findings are never ASK.
- FIX: one answer is clearly right from the files and code. Give the exact replacement text.

Reply as a list sorted by severity: [CRITICAL|HIGH|MEDIUM|LOW] [ASK|FIX] <file> <ID or section> — <issue>
— <consequence if left> — <fix or options>. ≤500 words.
```

### Spec checklist

```
The spec describes WHAT and WHY, not HOW. Only glance at the codebase to check a claim the spec makes.
- Gaps: core flows, error/edge cases, or user stories with no requirements or acceptance criteria
- Ambiguity: vague terms ("fast", "secure", "simple") with no measure; items open to two readings
- Contradictions: between requirements, goals vs non-goals, decisions vs constraints
- Requirements that aren't baseline properties (invariants or observable behaviors), or that conflict
  with or duplicate each other
- Untestable requirements or success criteria
- Scope: goals with no requirements; requirements that serve no goal; scope creep beyond the problem
- Implementation detail that doesn't belong (file paths, function names, schemas, library choices
  the user didn't require)
```

### Plan checklist

```
Spot-check the codebase. Verify that the files, functions, and commands the plan relies on exist and
behave as assumed (grep/read, don't read everything).
- Coverage: spec items (G, US acceptance criteria, R, E, SC) with no task; tasks that cite nothing; a coverage
  table that doesn't match the tasks
- References: missing **Spec:** link; spec ID citations that don't exist or are `*(removed)*`
- Spec drift: the plan contradicts the spec, or plans something the spec lists as a non-goal
- Wrong assumptions about the code: missing files/APIs, wrong paths, conventions ignored
- Parallel safety: units in the same wave owning the same files; missing or circular Depends on;
  a unit using something a later wave creates
- Executability: tasks too vague for an agent with no context; units too big or trivially small
- Verification: verify steps that can't run (missing/wrong commands); no tests for risky logic
- Missing pieces: migrations, config, error handling, backward compatibility, rollback for risky changes
```

## Step 2: Apply and report (main thread)

Sanity-check the findings yourself first. Drop anything that's really a nitpick, or whose stated consequence doesn't hold up. If a finding marked FIX is actually ambiguous, move it to ASK. If one marked ASK has an obvious answer, move it to FIX.

1. Apply every **FIX** directly. Don't run a second review of your own fixes.
2. Report what was auto-fixed, grouped by severity, one line each: `HIGH R3 — <what was wrong> → <what changed>`. If nothing was found, say so in one line.
3. Raise every **ASK** via `AskUserQuestion` (up to 4 per call, most severe first). Put the severity in the question, the recommended option first, and an "Ignore — leave as is" option. Skip this step if there are none. If a plan finding means the spec itself is wrong, say so and offer to fix the spec first.
4. For each answer, update the file to reflect the decision, recording notable ones under Key decisions. If the user ignores a finding, leave the file as is and remember it for later reviews in this conversation.

If you're mid-interview in `/rpi:arch` or mid-outline in `/rpi:plan`, fold the ASK findings into your next question round instead of asking them separately, then carry on.

Don't declare the spec or plan finished here. That's the user's call.
