# Sources and evidence

Evidence strength: **strong** (controlled study or replicated), **vendor** (official lab guidance from observed model behavior), **practitioner** (one experienced builder's report), **weak** (single preprint, anecdotal, or unreplicated). Most pre-2024 papers tested base or older models, so effect sizes on current frontier models are likely smaller even where the direction holds.

## Law 1 — Nothing in context is inert
- Examples teach format, label space, and input distribution more than correctness; random labels barely hurt. **strong** — Min et al. 2022, https://arxiv.org/abs/2202.12837
- Few-shot majority-label, recency, and common-token bias; accuracy swings with example choice and order. **strong (older models)** — Zhao et al. 2021, https://arxiv.org/abs/2102.09690 ; Lu et al. 2022, https://arxiv.org/abs/2104.08786
- Few-shot degrades RL-trained reasoning models; prefer zero-shot plus an explicit format. **weak (second-hand)** — DeepSeek-R1 guidance, https://docs.together.ai/docs/prompting-deepseek-r1
- Uniform action history makes agents brittle and repetitive; vary templates on purpose. **practitioner** — Manus, https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus
- Sample phrases get reused verbatim. **vendor** — OpenAI GPT-4.1 guide, https://developers.openai.com/cookbook/examples/gpt4-1_prompting_guide
- Negated instructions underperform; naming a forbidden token primes it. **weak** — NeurIPS 2022 workshop, https://neurips.cc/virtual/2022/65648 ; "Semantic Gravity Wells" 2026, https://arxiv.org/abs/2601.08070 (counter-evidence: https://arxiv.org/abs/2503.22395)
- Prompt formatting carries into output formatting. **vendor** — https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- Irrelevant context sharply reduces accuracy. **strong** — Shi et al. 2023, https://arxiv.org/abs/2302.00093
- Same-topic distractors hurt most; coherent filler masks targets more than shuffled filler. **strong** — Chroma Context Rot, https://research.trychroma.com/context-rot
- Context failure modes: poisoning, distraction, confusion, clash; splitting a prompt across turns lost 39% on average. **practitioner** — Breunig, https://www.dbreunig.com/2025/06/22/how-contexts-fail-and-how-to-fix-them.html
- Sycophancy is driven by preference training; models flip answers under pushback (~58%), and citation-backed pushback does the most damage. **strong** — Sharma et al. 2023, https://arxiv.org/abs/2310.13548 ; SycEval 2025, https://arxiv.org/abs/2502.08177

## Law 2 — Attention is a finite budget
- Context is a finite resource with diminishing returns; seek "the smallest set of high-signal tokens". **vendor** — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Degradation with length across 18 current models, even on trivial tasks; a focused ~300-token prompt far beat a ~113k one. **strong** — Chroma, https://research.trychroma.com/context-rot
- Lost in the middle (U-shaped recall by position). **strong (flatter on new models)** — Liu et al. 2023, https://arxiv.org/abs/2307.03172
- Effective context is far shorter than advertised; performance collapses without lexical overlap. **strong** — RULER, https://arxiv.org/abs/2404.06654 ; NoLiMa, https://arxiv.org/abs/2502.05167
- Instruction-following decays with count (best ~68% at 500); bias toward earlier instructions. **strong (simple tasks)** — IFScale, https://arxiv.org/abs/2507.11538
- Bloated CLAUDE.md makes Claude ignore instructions; "would removing this cause mistakes?"; /clear after two failed corrections. **vendor** — https://code.claude.com/docs/en/best-practices
- Skills: context is a public good; progressive disclosure; SKILL.md under 500 lines. **vendor** — https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
- Long documents first, query last: up to 30% better. **vendor** — Anthropic best practices (above)
- Keep context usage around 40–60%; incorrect info is worse than missing info, which is worse than noise. **practitioner** — HumanLayer ACE, https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md
- Recite the plan (todo.md) to keep goals in recent attention; make compression restorable by keeping paths and URLs. **practitioner** — Manus (above)
- Pointers need a when and a why or they get ignored. **practitioner** — https://blog.sshh.io/p/how-i-use-every-claude-code-feature ; https://www.humanlayer.dev/blog/writing-a-good-claude-md

## Law 3 — Tell it what it can't know
- Personas don't improve factual accuracy; the best persona is unpredictable. **strong** — Zheng et al. 2024, https://arxiv.org/abs/2311.10054 ; Wharton, https://papers.ssrn.com/abstract=5879722
- Threats and tips: no aggregate effect, large unpredictable per-question swings. **strong** — Wharton Report 3, https://arxiv.org/abs/2508.00614
- CoT gives marginal gains on reasoning models at 20–80% more time, and can hurt some tasks by up to 36 points. **strong** — Wharton Report 2, https://arxiv.org/abs/2506.07142 ; Mind Your Step, https://arxiv.org/abs/2410.21333
- "Default assumption: Claude is already very smart"; treat it as a brilliant new employee who lacks context. **vendor** — Anthropic skills and prompting docs (above)

## Law 4 — Reasons generalize
- The text-to-speech ellipsis example; "Claude is smart enough to generalize from the explanation." **vendor** — Anthropic best practices (above)
- "Give the reason, not only the request." **vendor** — https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5

## Law 5 — Emphasis over-triggers
- Dial back "CRITICAL: You MUST"; "If in doubt, use X" overtriggers. **vendor** — Anthropic best practices (above)
- Verification instructions cause over-verification; hedges are followed literally. **vendor** — https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5
- Skills written for older models are too prescriptive and degrade output. **vendor** — Fable 5 guide (above)
- "Be THOROUGH" caused tool overuse in GPT-5; ALWAYS/NEVER is a legacy habit, kept for true invariants. **vendor** — https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide ; https://developers.openai.com/api/docs/guides/prompt-guidance?model=gpt-5.5
- Counterpoint: one IMPORTANT on a single repeatedly-skipped line can still help. **vendor** — Claude Code best practices (above)

## Law 6 — Contradictions
- Contradictory prompts are "more damaging to GPT-5 than to other models"; small inclusions steer unexpectedly. **vendor** — OpenAI GPT-5 / 5.1 guides (above)

## Law 7 — Destination, not route
- Aim between rigid logic and vague guidance; heuristics over rigid rules. **vendor** — Anthropic context engineering and multi-agent posts (above)
- Match freedom to how fragile the task is; one default plus a way out. **vendor** — Anthropic skills best practices (above)
- "Describe the destination rather than every step"; start migrations from a fresh baseline. **vendor** — OpenAI GPT-5.5 guide (above)
- Removing 80% of an agent's tools raised success from 80% to 100% (small sample). **practitioner** — Vercel, https://vercel.com/blog/we-removed-80-percent-of-our-agents-tools

## Law 8 — Positive targets
- "Tell Claude what to do instead of what not to do." **vendor** — Anthropic best practices (above)
- Pair every "never X" with an alternative. **practitioner** — sshh (above)

## Law 9 — Examples
- 3–5 diverse, canonical examples rather than a laundry list of edge cases. **vendor** — Anthropic (above)
- Too many examples cause overfitting. **vendor** — Gemini, https://ai.google.dev/gemini-api/docs/prompting-strategies
- Examples collapsing output diversity: plausible, but no direct controlled study found. **weak**

## Law 10 — Trust, and verify with signals
- Intrinsic self-correction doesn't improve reasoning without external feedback. **strong** — Huang et al. 2024, https://arxiv.org/abs/2310.01798
- Models can't locate their own errors but fix them well once told where. **strong** — Tyen et al. 2024, https://arxiv.org/abs/2311.08516
- LLM judges are biased toward their own outputs, by position, and by verbosity. **strong** — https://arxiv.org/abs/2404.13076 ; https://arxiv.org/abs/2306.05685
- Fresh-context verifier subagents outperform self-critique. **vendor** — Fable 5 guide (above)
- Current models verify unprompted; remove "double-check" instructions. **vendor** — Opus 5 guide (above)
- Review the plan, not the code: errors in research and plans compound. **practitioner** — HumanLayer ACE (above)
- Design agentic loops around clear success tests, sandboxes, and budget-capped credentials instead of human checkpoints. **practitioner** — https://simonwillison.net/2025/Sep/30/designing-agentic-loops/

## Law 11 — Delegation
- Each subagent needs an objective, an output format, tool guidance, and boundaries; vague briefs duplicated work. **vendor** — https://www.anthropic.com/engineering/multi-agent-research-system
- Don't delegate what a handful of tool calls can do; current models over-spawn subagents. **vendor** — Opus 5 guide (above)
- Parallel agents make conflicting implicit decisions. **practitioner** — Cognition, https://cognition.com/blog/dont-build-multi-agents
- Many parallel readers, one builder. **practitioner** — Huntley, https://ghuntley.com/ralph/

## Law 12 — Tools
- Distinct purposes, namespacing, names over UUIDs, capped output, actionable errors, concise response modes. **vendor** — https://www.anthropic.com/engineering/writing-tools-for-agents
- Design tools so they're hard to misuse; more time went into tools than into the prompt. **vendor** — https://www.anthropic.com/engineering/building-effective-agents
- Don't add or remove tools mid-task (KV-cache, dangling references); mask them instead. **practitioner** — Manus (above)
- Agent tools should be fast and quiet, crash loudly, and log to files the agent can read. **practitioner** — Ronacher, https://lucumr.pocoo.org/2025/6/12/agentic-coding/

## Unverified or weak, flagged
- XML being superior to other delimiters: vendor guidance only, no rigorous evidence.
- HumanLayer's reading that ~150–200 instructions is the practical ceiling: not checked against the paper.
- A GPT-6 guide's claim that over-specific guidance hinders results: page not reachable.
