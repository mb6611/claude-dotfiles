---
description: Implement stage — execute an approved plan with parallel subagents in dependency waves
argument-hint: "<plan slug or path> [--wave N] [--serial]"
---

You are in the **IMPLEMENT stage** of the spec → plan → implement cycle. You are the **orchestrator**: you schedule work, verify it, and keep the plan up to date. Subagents write the code.

User input: $ARGUMENTS

## Step 1: Preflight

1. Read the master plan `.ru/plans/<slug>.plan.md`, every subplan in `.ru/plans/<slug>/`, and the matching `.ru/specs/<slug>.spec.md`. If `$ARGUMENTS` is empty, pick the plan that has unchecked tasks (ask if more than one).
2. Running `/rpi:implement` means the user considers the plan done. If it still has open questions or TODOs, raise them before starting.
3. **Resume**: trust existing `[x]` checkmarks and start from the first unchecked task.
4. Check `git status`. If there are unrelated uncommitted changes, mention them before you start. Don't create branches or commits unless the user asks.

## Step 2: Schedule

Build the execution waves from the plan's waves table and each unit's **Depends on** and **Owns** lines. Then size the subagents, aiming for the **fewest agents that still parallelize the work**:

- **Tiny plan** (one unit, or fewer than ~6 tasks total): don't spawn anything. Implement it yourself, sequentially.
- **One subagent per unit** by default.
- **Merge** small units (fewer than ~3 tasks) that are in the same wave and have related files into one agent.
- **Split** a large unit (more than ~12 tasks) across 2–3 agents, but only along disjoint file sets within that unit.
- Two agents in the same wave must never own the same file. If ownership overlaps, run them in sequence, or as a last resort give one of them `isolation: "worktree"` and merge afterwards.
- Run at most ~5 agents at once. Queue the rest into the next round of the same wave.
- Use `model: "sonnet"` for clearly mechanical units (boilerplate, renames, straightforward tests). Leave the model unset for anything that needs judgment.

Show the user the schedule in a compact table (wave → agents → units → files) and proceed. Pause for confirmation only if something is unusual, such as overlapping ownership or a very large fan-out. With `--serial`, run units one at a time. With `--wave N`, run only that wave.

## Step 3: Execute wave by wave

Spawn every agent in a wave **in a single message** so they run concurrently. Each agent gets a self-contained prompt; it does not see this conversation:

```
You are implementing unit <UN name> of an approved plan.

Plan: <path to subplan, or master plan + "UN">. Read it first.
Spec: <spec path>. Read only the items your tasks cite (<R1, E2, …>).

You own these files, and only these: <list/globs>.
If you need to change anything outside them, STOP and report what and why. Do not work around it.

Do:
- Implement the unchecked tasks in your unit, in order, following existing code conventions.
- Run your unit's automated success criteria and fix failures until they pass.
- Do NOT edit any .plan.md or .spec.md file. The orchestrator tracks progress.
- If the code doesn't match what the plan assumes, stop and report: expected / found / why it matters.

Reply in ≤150 words:
- DONE: task IDs completed (UN.x)
- FILES: files changed
- VERIFY: commands run + pass/fail
- ISSUES: deviations, blockers, or follow-ups (or "none")
```

After each wave:
1. Read the reports. Don't re-read whole files unless something looks off. Spot-check with `git diff --stat` and targeted reads.
2. Run the project-wide automated checks (build, typecheck, tests, lint) once. If integration broke, fix small things yourself, or spawn one focused fixer agent with the error output.
3. Tick the completed tasks and passing automated criteria in the plan files. Leave manual criteria unchecked.
4. If any agent reported a mismatch or blocker, **stop** and bring it to the user before starting the next wave: what was expected, what was found, and options. If one parallel unit failed but others succeeded, keep their work and continue only with waves that don't depend on the failed unit.

## Step 4: Verify against the spec

When all waves are done, spawn **one** fresh read-only verifier subagent (`subagent_type: "Plan"` or `general-purpose`) with the spec path, the plan path, and `git diff` scope. Ask it to check each spec requirement (R), edge case (E) and success criterion (SC) against the implementation, and to report only gaps or regressions, citing spec IDs and file:line references, in ≤300 words. Fix real gaps, or raise them to the user if they need a decision.

## Step 5: Wrap up

1. Summarize: what was built, verification evidence (actual command results), deviations from the plan, and the **manual** checks the user needs to do.
2. After the user confirms the manual checks, tick them.
3. Don't commit. Offer `/git:commit`.
