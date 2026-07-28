# CV Version Comparison Methodology

> **Context:** When multiple AI agents produce CV versions for the same role, or when the user edits your version, a comparison analysis identifies what to keep, fix, and merge.

## When to Compare

- You produced a CV, user produced another version (from another AI or manual edits)
- Two CVs exist for the same role and you need to pick or merge
- User asks "how does yours compare to this one?"

## Comparison Dimensions

Score each dimension winner:

| Dimension | What to Check |
|-----------|---------------|
| JD Tailoring | Does the CV mirror the job description's exact language? Does it name their vendors/tools/systems? |
| Error Rate | Typos, hallucinated facts, impossible claims |
| Density | Facts per line. No filler words. |
| Structure | Does it follow the canonical compact rules (max 3 pages, compound bullets, Key Achievement)? |
| Framing | Does it turn negatives into positives? (e.g., Belgrade location → "strategic advantage for your Serbian jurisdiction") |
| Coverage | Are all significant roles represented? |

## Merging Decision Matrix

| Scenario | Action |
|----------|--------|
| One CV clearly wins on most dimensions | Pick the winner, patch its specific errors |
| Each wins on different dimensions | Merge: use the stronger structure as base, inject winning lines from the other |
| Both flawed | Start fresh, stealing the two best lines from each |

## What to NEVER Keep

- Typos (`Overseaw` → `Oversaw`)
- Hallucinated tool/model names you cannot verify
- Verbose paragraphs that violate the 3-page limit
- Standalone sections the user has explicitly removed (Case Studies, Certifications, etc.)

## Real Example: Hermes vs Gemini 3.1 Pro (Reluna Head of Operations)

**Gemini won on:** JD tailoring (named LSEG/Refinitiv, Stripe), location framing ("strategic bridge between Cyprus and Serbia"), more aggressive operational language ("Commanded high-volume FinTech operations")

**Hermes won on:** Zero errors, better coverage (kept Algotech), cleaner structure following canonical rules

**Verdict:** Merge. Use Hermes structure as base. Inject Gemini's header line, summary phrasing, vendor name-checks, and location paragraph. Fix Gemini's typo. Verify or delete the hallucinated "Ollama Minimax 2.5" claim.
