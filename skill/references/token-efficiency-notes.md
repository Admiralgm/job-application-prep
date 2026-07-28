# Token Efficiency & Model Performance Observations

> Session: 2026-05-24. User burned ~200M tokens on Deepseek V4 Pro in a few days.
> Observations about token efficiency and model selection.

## Token Burn Is Not Just About Workload

Token consumption varies significantly by model and approach:

1. **Context window tax**: Deepseek V4 Pro has a 1M token context. Every multi-turn call carries full conversation history. After 20+ turns, you may process 500K+ tokens of history per call.

2. **Output verbosity varies by model**: Deepseek tends to be thorough — explains reasoning, provides context. That's quality but expensive. Same content in 200 vs 800 tokens is a 4x cost difference. Over hundreds of calls, this compounds brutally.

3. **Profile KB loading**: Loading the full 331-line profile master summary into every call is expensive. Load only relevant sections per task when possible.

4. **Batching saves tokens**: One big call with all questions burns less than five small calls that each reload the same context.

## Deepseek V4 Pro — Performance Assessment

The user is genuinely impressed. 200M tokens burned in a few days of intensive use:
- Multi-model CV bake-offs (Reluna, refurbed)
- Job-hunting engine (scraping + benchmarking + generating + humanizing)
- Application form answers (humanized)
- Board presentation simulations

SWE-bench 80.6% manifests in practice — produces work that Claude Opus 4.5 reviews and calls "the stronger document."

**Limitation**: Suspended peak hours (14:00-21:00 CET weekdays). Plan heavy reasoning for early morning/late evening.

## Model Selection Heuristic

| Task | Best Model | Why |
|------|-----------|-----|
| Complex reasoning, research synthesis | Deepseek V4 Pro | 80.6% SWE-bench, thorough output |
| JD language mirroring, vendor naming | Gemini 3.1 Pro | Best at reading and echoing JD vocabulary |
| Risk-spotting, editorial review | Claude Opus 4.5 | Best at catching gaps and structural issues |
| Scraping, general analysis, long-context | OWL Alpha | Free, 1M context, always available |
| CV baseline + factual accuracy | Hermes (me) | Grounded in verified profile data |

## The Job-Hunting Engine

Built on Deepseek V4 Pro via Hermes + Ollama Cloud:
- Scrapes vacancies from multiple portals (bypasses anti-bot)
- Benchmarks against profile KB (built from 100s of past CVs/cover letters)
- Generates tailored applications with humanizer
- ~5M tokens per full application cycle (scrape + score + CV + cover letter + 5 form answers + humanize)
- User's next phase: auto-fill bot that runs while he drinks coffee

This is a real production system, not a toy. The 200M token burn reflects genuine intensive use.
