---
name: curate
description: Decides when something from the conversation should change the user's persistent memory, instructions, skills, or commands, and where it belongs. Use whenever the user corrects you, states a preference ("btw", "stop", "always", "in general"), types a request they've made before, explains a concept of their own, or calls something useless or unnecessary, and at the end of a task to check whether anything durable came up. Also use when the user asks to remember or save something, update a skill, command, or CLAUDE.md, or asks where a rule should live.
---

# Curating what Claude remembers

Everything saved is context a future session pays for. A good save prevents a mistake the next session would otherwise make. A bad save costs attention in every session, gets copied as if it were true, and crowds out the rules that matter. So the default is to save nothing. Save only when the principles below say it's worth it, put it in the one place it belongs, and remove what no longer earns its place.

## Principles

### 1. Saved context is never free
Always-loaded text dilutes attention, and instruction-following decays as rules pile up. Agent-written instruction files have been measured to lower task success and raise cost when they restate what the repo already says.
- Save only what a future session can't work out on its own: the user's preferences, their own concepts, local facts not visible in the code, and lessons that cost a mistake to learn.
- Don't save task details, things derivable from code or git history, explanations given for the user's understanding, or anything already in a skill or command.
- For always-loaded files, every line must pass "would removing this cause a mistake?"

### 2. Save only on evidence from outside your own judgment
A lesson you inferred and never checked is a guess, and a saved guess gets followed as fact. Evidence that counts:
- **The user said it**, as a correction or a stated preference. Their preferences are authoritative.
- **Repetition.** They asked for the same thing by hand twice.
- **A verified result.** Tests pass, the tool behaves as claimed, or the fix worked on real output.

A user's *factual* claim about code or tools is not evidence by itself. Check it before saving, because agreeing under pushback and then saving the agreement makes the mistake permanent.

### 3. Scope a lesson to where it came from
A correction made in one context, saved as a universal rule, misfires everywhere else. Ask where it applies: everywhere, this repo, one workflow, or only this task. Save at that scope and no wider. Only task-scoped means don't save.

### 4. Kind and scope decide the home
Each lesson lives in exactly one place. Pick it by what the lesson *does*, not by the word the user used ("memory", "skill", "rule"):

| What it is | Where it goes |
|---|---|
| How Claude should behave everywhere (style, tools, preferences) | `plugins/memory/CLAUDE.md` in the dotfiles repo, which loads every session on every machine |
| A fact or rule for one repo | That repo's `CLAUDE.md` or `.claude/` |
| A rule for one workflow | That workflow's command or skill (e.g. `plugins/rpi/commands/arch.md`) |
| A lesson about writing prompts, skills, or instructions | The `prompting:effectively` skill, following its own update steps |
| A multi-step procedure the user repeats | A command in the plugin it fits; a new plugin only if none does |
| Something that must be enforced, not just suggested | Settings or a hook, kept in the dotfiles repo where possible |

The dotfiles repo is `~/Developer/personal/claude-dotfiles`. Don't put a user-wide lesson in the per-project auto-memory directory, because it only loads in that one project on that one machine. Use auto-memory only for things specific to the current project.

Edits to skills, commands, and instruction files change behavior for every future session, so they need more evidence than a memory does (principle 6).

### 5. Reconcile, then edit in small pieces
Before writing, read the target and decide: **add** (new), **update** (refines an existing line), **retire** (contradicted or obsolete, so delete it, since git keeps history), or **no-op** (already covered).
- Make the smallest edit that works. Never regenerate a whole file. Full rewrites drift toward generic text and silently lose detail.
- Write the cleaned-up rule with its reason in the same sentence, so it generalizes. Never store the user's raw wording or frustration.
- Before adding a skill or command, check for one that overlaps and extend it. Look-alike skills steal each other's triggers.
- Removing is as valuable as adding. When the user calls something dumb, useless, or unnecessary now that models are better, or asks for a native way, delete the scaffolding instead of patching it.

### 6. Act on clear signals, propose the rest at a natural break
- **Act, then say so in one line**, when the user explicitly states a durable preference or correction ("btw use pnpm", "stop saying X") and the home is clear. Example: "Saved to `plugins/memory/CLAUDE.md`: use pnpm for JS/TS projects."
- **Propose in one line and wait for a yes** for anything you inferred, any new skill or command, and any change to a workflow's rules or output format. When the change affects what a workflow produces, first apply it to the real output in front of the user so they can judge it, then offer to update the rule.
- Don't interrupt mid-task with proposals. Batch them at the end of the task.
- Don't commit. Mention that `/save` will commit and push the dotfiles repo.

## Signals in this user's conversations
These are how durable lessons have actually shown up. They're signals to weigh, not triggers to fire on:
- A correction with a "from now on" marker: "btw", "stop", "always", "in general", "should just".
- Spotting a problem in a workflow's real output ("isn't this kind of unreadable?"), then asking to see the fix there first.
- Typing the same multi-step request by hand a second time.
- A long explanation of a concept the user named themselves.
- Asking whether something will carry over to a new machine.
- Calling something dumb, useless, or unnecessary, or asking for the native option.

See [references/cases.md](references/cases.md) for real past cases and where they belong.

## Keeping this skill current

This skill is a living record. Update it when curation goes wrong or a new pattern appears:
1. **The user undoes or objects to a save, or points out something you should have saved.** Find the principle it falls under and sharpen that bullet, or adjust the signals or table. Add the case to `references/cases.md` in one line: what happened, and the verdict.
2. **New research or a credible source** changes a principle: update the principle and add the source to [references/sources.md](references/sources.md) with its evidence strength.
3. Prune as you add. Delete guidance a new lesson makes obsolete, and keep this file under ~150 lines. It must follow its own principle 1.
