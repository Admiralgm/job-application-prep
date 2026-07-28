# Reluna Head of Operations — Multi-Model CV Comparison (2026-05-24)

## What happened
Three models produced/reviewed a CV for the Reluna Head of Operations role:
- **Hermes** (me): Produced the baseline CV
- **Gemini 3.1 Pro**: Produced an alternative CV
- **Claude Opus 4.5**: Reviewed both and rendered verdict

## Model strengths observed

### Hermes
- Factual accuracy — no hallucinations
- Quantitative density (€5.2M, 27K patients, 127 staff, $500M+)
- Role completeness (kept UNICEF GIGA + Algotech — both absent from Gemini's version)
- Canonical style adherence

### Gemini 3.1 Pro
- JD language mirroring — named LSEG/Refinitiv and Stripe (the JD's exact vendors)
- Strategic location framing — "uniquely positioned to manage Reluna's Serbian jurisdiction on the ground while executing a high-frequency hybrid presence in Nicosia, Cyprus"
- Stronger verbs: "commanded" instead of "led"
- **Hallucinated** "Ollama Minimax 2.5 cloud configurations" — must always fact-check Gemini's AI-specific claims

### Claude Opus 4.5
- Risk-spotting: flagged Olivia Education (March 2026–Present, 2 months in) as potential "job hopper" risk → led to contract-type rule
- Gap analysis: flagged UAE/Cyprus jurisdictional experience gap as biggest CV risk
- Structural critique: correctly identified Hermes' version as more scannable

## Final merged CV
`~/CV_REPOSITORY_DATABASE.md`

## Key takeaways for future multi-model CV generation
1. Build Hermes baseline first (structure + accuracy)
2. Inject Gemini's JD-mirroring and strategic framing
3. Apply Claude's risk review (recent roles, gaps, structure)
4. Always fact-check Gemini's specific technical claims
5. The synthesis method is: Hermes content → Gemini tailoring → Claude risk review → merged output
