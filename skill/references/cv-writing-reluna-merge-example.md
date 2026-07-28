# Multi-Model CV Synthesis — Worked Example: Reluna Head of Operations

> Session: 2026-05-24. Three-model bake-off for the Reluna Head of Operations role.
> Hermes produced the baseline CV. Gemini 3.1 Pro produced a competing version. Claude Opus 4.5 reviewed both.

## The Three Versions

### Hermes v1 (Baseline)
- 61 lines, structured, all roles covered
- Strong quantitative anchors (€5.2M, 27K patients, 127 staff, $500M+)
- Weakness: generic location framing, less aggressive JD language mirroring

### Gemini 3.1 Pro (Competitor)
- Denser prose, fewer roles (dropped Algotech entirely)
- Strengths: named JD vendors (LSEG/Refinitiv, Stripe), reframed location as strategic advantage
- Best line: "Uniquely positioned to manage Reluna's Serbian jurisdiction on the ground while executing a high-frequency hybrid presence in Nicosia, Cyprus."
- Weaknesses: hallucinated "Ollama Minimax 2.5 cloud configurations", typo "Overseaw", dropped roles

### Claude Opus 4.5 (Reviewer)
- Verdict: "Hermes CV is the stronger document"
- Key feedback: contract-type rule for recent roles, Olivia Education flagged as risk
- Gap identified: neither CV addressed the Cyprus/UAE jurisdictional experience gap

## Merge Decisions

| Element | Source | Decision |
|---------|--------|----------|
| Header location line | Gemini | "Ready for Hybrid Nicosia Operations" |
| Summary opening | Gemini | "C-level Operations Executive" + "strategic bridge" |
| Summary bullet 2 | Gemini | "Commanded high-volume FinTech platforms" |
| Summary bullet 4 | Gemini | "uniquely positioned to manage Reluna's Serbian..." |
| Competencies | Gemini | Added LSEG/Refinitiv, Stripe to vendor line |
| Olivia role title | Claude | Added "(Project-Based / Advisory)" |
| Location section | Gemini | Full paragraph reframing geography as strategy |
| All quantitative anchors | Hermes | Kept all |
| UNICEF + Algotech | Hermes | Kept both (Gemini dropped them) |

## Lessons

1. Always triangulate for high-stakes roles.
2. Gemini: best at JD mirroring, vendor naming, location framing.
3. Claude: best at risk-spotting, structural critique.
4. Hermes: best at factual accuracy, quantitative density.
5. Always verify Gemini's technical claims (hallucinated "Minimax 2.5").
6. Location is strategy, not logistics.
7. Name the JD's vendors in Competencies.
