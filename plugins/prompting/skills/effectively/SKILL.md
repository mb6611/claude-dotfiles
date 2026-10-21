---
name: effectively
description: First-principles guide to prompting and context engineering for current models. Use whenever you write or edit text another model will read, such as a CLAUDE.md, skill, slash command, subagent brief, system prompt, tool description, or eval prompt, and when reviewing or debugging why a prompt misbehaves. Also use when the user shares a new prompting lesson to record.
---

# Prompting effectively

A model is a function from context to a distribution over next tokens. It is already highly capable. Your prompt doesn't teach it to think. It conditions that function. Every principle below follows from that, so when a case isn't covered, reason from the laws rather than looking for a rule.

## The laws

### 1. Nothing in context is inert
Every token shifts the output distribution, including tokens you meant as background, as warnings, or as "just an example".
- **Examples become templates.** Models copy the format, length, label distribution, and phrasing of examples more than their logic. A single example collapses the space of answers toward itself.
- **Naming a thing primes it.** "Don't mention X" puts X in play. This is worst when X is what the model would naturally say.
- **Style is contagious.** A prompt full of headers and bullets gets headers and bullets back. Write the prompt in the style you want back.
- **Irrelevant context distracts, and wrong context poisons.** Plausible but off-topic material hurts more than obvious noise. An error written into a goal or summary gets reused as fact. Delete stale and wrong content; don't argue with it.
- **Framing anchors.** Leading with your proposed answer biases the model toward agreeing. Ask neutrally when you want an independent judgment.

### 2. Attention is a finite budget that degrades with length
Quality falls gradually as context grows, well before the window is full, and it falls fastest for facts that don't share words with the question.
- Aim for the smallest set of high-signal tokens that gets the outcome. For every line, ask: *would removing this cause a mistake?* If not, cut it.
- Instruction-following decays as instructions pile up, and adding more degrades all of them, not just the new ones. A rule that keeps getting broken usually means the prompt is too long, not too quiet.
- Load context just in time. Keep pointers (paths, URLs, queries) and fetch when needed. Give a pointer a *when* and a *why*, or it gets ignored.
- Position matters. Put long documents first and the question or task last. Keep the live plan near the end of context, for example by rewriting a todo list.
- Fresh context beats piled-up corrections. After a couple of failed fixes, restart with a better prompt instead of adding turn-by-turn patches.
- Isolate exploration in subagents. They can spend tens of thousands of tokens and return one or two thousand.

### 3. Tell it what it can't know, not what it already knows
The model already knows general best practices, how to reason, and how to be careful. What it can't know is your situation. Spend tokens only on that:
- the goal and who the output is for, and what it enables
- local facts: conventions, constraints, where things are, what was already tried
- what "done" looks like

Old tricks that supplied nothing are now noise or worse: personas ("you are an expert…"), "think step by step" for reasoning models, tips and threats, generic advice ("write clean code", "handle errors gracefully"). Treat the model as a brilliant new colleague who lacks context, not as someone who lacks skill.

### 4. Reasons generalize, rules don't
A bare rule covers the cases you listed. A reason covers the cases you didn't. "Never use ellipses" is brittle. "This will be read by a text-to-speech engine, so avoid ellipses" lets the model extrapolate correctly. When you add a constraint, add the why in the same sentence.

### 5. Emphasis is relative, and current models take you literally
Older models needed shouting. Current ones follow instructions closely, so loud language over-triggers.
- "CRITICAL: You MUST use X" makes the model use X where it shouldn't. "If in doubt, do X" makes it do X constantly. Write "Use X when …".
- Keep ALWAYS / NEVER / MUST for true invariants (safety, required output fields). If many lines are emphasized, none stand out.
- Qualifiers are followed literally. "Only report severe issues" will suppress real but moderate ones. Say exactly what you want.

### 6. Contradictions cost more than omissions
A smarter model tries harder to satisfy every instruction, so two instructions in tension produce hedging or erratic behavior. Each fact and rule lives in one place. When you edit a prompt, look for what the new line now conflicts with.

### 7. Describe the destination, not the route
Specify the outcome, its constraints, and how to tell it's done. Then let the model choose the steps. How much freedom to give depends on how easy the task is to break: exact steps when one wrong move causes damage (a migration, a release script), only direction when many approaches work (design, research, writing). Aim between rigid if-else rules and vague encouragement. Give one sensible default plus a way out over a menu of options.

### 8. Point at the target, not away from it
"Write flowing prose paragraphs" works better than "don't use markdown". Every prohibition needs an alternative, or the model either stalls or drifts toward the forbidden thing. This is law 1 applied to negation.

### 9. Use examples deliberately
Examples are the strongest signal in a prompt (law 1), so treat them like a dangerous tool.
- **Use them** when the target is easier to show than to describe: an exact output format, a schema, a tone.
- **Avoid them** for open-ended, judgment, or creative tasks. They narrow the space toward the example. Describe the properties you want instead.
- **When you use them:** 3–5, deliberately varied along the dimensions that should vary. Label them as illustrative ("these show the quality bar, not content to copy"). Never use only one example for a task with many valid answers.
- In long agent loops, vary templates and phrasing in your own scaffolding so the agent doesn't start repeating itself.

### 10. Trust the model, and check work with real signals, not habit
Current models check their own work unprompted. Many human checkpoints and "double-check your answer" instructions are habits from weaker models, and they now waste tokens or cause over-verification.
- **Remove** reflexive verification instructions and confirmation checkpoints that exist out of habit. Let the agent act, keep going, and finish.
- **Self-critique alone doesn't find errors.** Verification works when it has an outside signal: tests, type checkers, tools, real output, or a fresh-context reviewer that didn't produce the work.
- **Make the environment verifiable** instead of supervising it: runnable tests, quiet and fast tools, logs the agent can read, sandboxes for risky actions.
- **Spend human attention where errors compound.** A wrong spec or plan corrupts everything downstream, so review those. Don't review every step of the code that follows from them.
- Keep humans in the loop for irreversible or outward-facing actions and genuine decisions of taste or priority, not for things the model can check itself.

### 11. Delegation briefs are prompts with zero shared context
A subagent sees only its brief. Give it the objective, why it matters, its boundaries (what it owns and what it must not touch), and the exact output format and size. Vague briefs produce duplicated work. Don't delegate what takes a handful of tool calls. Parallel agents that each make implicit decisions will conflict, so parallelize only work that is truly separable (reads, disjoint files).

### 12. Tools are prompts too
Tool names, descriptions, parameters, outputs, and errors all land in context.
- Give each tool one distinct purpose and a minimal tool set.
- Return names rather than opaque IDs, and keep output capped and filterable.
- Write errors that say how to fix the call.
- Don't add or remove tools mid-task. It invalidates the cache and confuses earlier references.

## Writing and revising a prompt

1. State the goal, the audience, and what done looks like in plain words. If you can't, the prompt isn't the problem yet.
2. Write what the model can't know (law 3), with reasons (law 4), in the style you want back (law 1).
3. Cut. Every line must pass "would removing this cause a mistake?"
4. Check for contradictions (law 6), loud emphasis (law 5), bare negations (law 8), and examples that will be copied (law 9).
5. Test it empirically. Change one thing at a time. When moving to a newer model, start from a fresh minimal baseline rather than porting old scaffolding, because prompts written for weaker models are usually over-prescriptive for stronger ones.

## Keeping this skill current

This skill is a living record. When the user shares a prompting lesson, or you find one from a credible source:
1. Find the law it follows from, and add it there as one short bullet. If it truly fits none, that may be a new law. Say so to the user instead of forcing it.
2. Add the source and its evidence strength to [references/sources.md](references/sources.md).
3. Remove anything the new lesson makes obsolete. Model behavior changes between generations, so older guidance should be pruned, not stacked.
4. Keep this file well under 500 lines. It must obey its own law 2.
