# Sources and evidence

Evidence strength: **strong** (controlled study or replicated), **vendor** (official guidance from the tool's builders), **practitioner** (one experienced builder's report), **weak** (single preprint or unreplicated). Papers from 2023–2024 tested older models; the mechanisms hold, the thresholds have moved.

## 1. Saved context is never free
- Context is a finite attention budget; aim for the smallest set of high-signal tokens. **vendor** — Anthropic, https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Performance falls as input grows, even on easy tasks; ~300 focused tokens beat ~113k. **strong** — Chroma Context Rot, https://www.trychroma.com/research/context-rot
- Instruction-following decays with instruction count (98% at 100, 69% at 500 for the best model). **strong (simple tasks)** — IFScale, https://arxiv.org/abs/2507.11538
- LLM-generated AGENTS.md lowered success and raised cost 20–23%; human-written ones helped ~4%, not at all for Claude Code. **strong (Python only, 138 tasks)** — Gloaguen et al. 2026, https://arxiv.org/abs/2602.11988
- CLAUDE.md is context, not enforced config; under 200 lines; contradictions make Claude pick one arbitrarily. **vendor** — https://code.claude.com/docs/en/memory
- Hand-write CLAUDE.md, keep it short, prefer pointers to copies. **practitioner** — HumanLayer, https://www.humanlayer.dev/blog/writing-a-good-claude-md
- Progressive disclosure: descriptions always loaded, bodies on demand. **vendor** — Anthropic, https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills

## 2. Save only on outside evidence
- Self-correction without an external signal doesn't improve, and sometimes hurts. **strong (older models; still holds per 2026 trace study)** — Huang et al. 2023, https://arxiv.org/abs/2310.01798 ; https://arxiv.org/abs/2601.00514
- Self-correction works with reliable outside feedback (tests, tools). **strong (survey)** — Kamoi et al. 2024, https://arxiv.org/abs/2406.01297
- Skills a model writes without experience: −1.3pp; curated ones: +16.2pp, on current frontier models. **strong** — SkillsBench 2026, https://arxiv.org/abs/2602.12670
- Only verified skills enter the library; removing verification cut progress ~73%. **strong** — Voyager, https://arxiv.org/abs/2305.16291
- Saving every experience was the worst policy; a strict save judge plus deletion was best. **weak (single preprint)** — Xiong et al. 2025, https://arxiv.org/abs/2505.16067
- Preference training makes models cave to pushback; saving the capitulation makes it permanent. **strong (older models)** — Sharma et al., https://arxiv.org/abs/2310.13548

## 3. Scope a lesson to where it came from
- Preferences inferred per context beat one global preference; the global version did worse than no learning. **weak (simulated user)** — CIPHER, https://arxiv.org/abs/2404.15269

## 4. Kind decides the home
- Working / episodic / semantic / procedural memory; writing to procedural memory is riskier. **strong (taxonomy)** — CoALA, https://arxiv.org/abs/2309.02427

## 5. Reconcile, then edit in small pieces
- Full rewrites collapse context (18,282 → 122 tokens, below baseline); itemized delta edits with counters fix it. **strong** — ACE, https://arxiv.org/abs/2510.04618
- ADD / UPDATE / DELETE / NOOP against the most similar existing memories. **weak (disputed benchmarks)** — Mem0, https://arxiv.org/abs/2504.19413
- Lessons need to be voted down and deleted, since even successes can be suboptimal. **strong** — ExpeL, https://arxiv.org/abs/2308.10144
- Mark facts invalid instead of erasing them, so history is kept. **weak** — Zep, https://arxiv.org/abs/2501.13956
- Large or look-alike skill libraries cut pass rates up to 21%; shadowing explains most of it. **weak (single preprint, current Claude models)** — Song & Wei 2026, https://arxiv.org/abs/2605.24050

## 6. When to write
- Writing in the hot path (immediate, adds task load) vs in the background (no load, needs a trigger). **practitioner** — LangChain, https://www.langchain.com/blog/memory-for-agents
- Reflect when enough has accumulated, not on every event; keep links to evidence. **strong (original design, older models)** — Generative Agents, https://arxiv.org/abs/2304.03442
- Consolidating between turns cuts later cost when future queries are predictable. **weak** — Sleep-time compute, https://arxiv.org/abs/2504.13171
