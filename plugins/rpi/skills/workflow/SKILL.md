---
name: workflow
description: The spec → plan → implement (RPI) workflow. Use whenever the user asks you to build, add, or redesign a feature that is more than a small change, even if they don't mention specs, so you start in spec mode instead of writing code. Also use whenever the project has a .claude/specs/ or .claude/plans/ directory, when reading or editing any *.spec.md or *.plan.md file, or when the user talks about a feature that has a spec or plan. Tells you which stage the user is in, what is allowed in that stage, and when to review specs and plans on your own.
---

# RPI workflow

## Default to spec mode for real features

When the user asks for a feature of sufficient size, don't start writing code. Enter the Spec stage and follow `rpi:arch`, even if they didn't ask for a spec. Tell them in one line that you're starting with a spec. If they explicitly say to skip it, skip it.

A feature is sufficient when any of these is true: it adds new user-facing behavior, touches several files or modules, involves design decisions or tradeoffs, or you'd have questions before you could build it well.

These don't need a spec: bug fixes with a clear cause, small tweaks, renames, config changes, one-file edits, or anything whose whole diff you could describe in a sentence or two. Just do them.

## Stages

Features move through three stages, each with its own files:

| Stage | Command | Files | Allowed to change |
|-------|---------|-------|-------------------|
| Spec | `/rpi:arch` | `.claude/specs/<slug>.spec.md` | only spec files |
| Plan | `/rpi:plan` | `.claude/plans/<slug>.plan.md` (+ `.claude/plans/<slug>/NN-<unit>.plan.md` subplans) | only plan files (and the spec, with the user's agreement) |
| Implement | `/rpi:implement` | source code; plan checkboxes | code, plus progress in plans |

## Know the stage

There's no status metadata. Work out a feature's stage from which files exist for its slug and the plan's checkboxes. Use `ls .claude/specs .claude/plans` and skim the plan. Do this once, and again if things change, not every turn.

- spec only → **Spec stage**. Help refine the spec. No code, no plan.
- plan exists, no tasks ticked → **Plan stage**. Refine the plan. No code.
- plan partly ticked → **Implement stage**. Resume from the first unchecked task.
- everything ticked → finished. Treat changes as a new iteration.

A stage ends when the user says so, usually by running the next command. Never decide on your own that a spec or plan is done. If the user asks for something that skips a stage, such as writing code while still in the Spec stage, say which stage they're in and ask before skipping.

## Review on your own judgment

After you edit a spec or plan, decide whether it now needs a review. Invoke `rpi:review` on it when the change is **significant**:
- a first complete draft now exists
- requirements, user stories, scope, units, dependencies, or file ownership were added, removed, or reworked
- an answer from the user changed the direction of the feature
- several rounds of smaller edits have built up since the last review

Skip the review for small things: typo and wording fixes, a single clarification folded in, ticking checkboxes, or edits that a review itself just made. Don't run a review on a review's own fixes.

When you're mid-interview, finish the current question round first. Then fold any ASK findings from the review into your next questions instead of interrupting.
