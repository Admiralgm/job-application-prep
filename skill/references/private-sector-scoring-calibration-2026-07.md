# Private-Sector Scoring Calibration — 2026-07

## Discovery

Two pure private-sector roles scored on 2026-07-24 exposed a P3 systematic
error in the V5.0.0 scoring engine SKILL.md (`vaccancy-compatibility-scoring-engine`).

## The Problem

The P3 (UN/IFI/Development Fit) parameter table in the scoring engine SKILL.md
has NO row for pure private-sector roles. The table covers UN agencies, IFIs,
INGOs, bilateral aid, and "other development orgs" (8-10) — but nothing for
tech services, iGaming, consulting, CPG, or other non-development private sector.

Without guidance, the scorer defaults to 0-2 for private sector, which is
**systematically too low** by 3-6 points compared to the calibration established
in the scoring engine's own `pmi-private-sector-scoring-pattern-2026-06-09.md`.

## Evidence

### This session — two private-sector roles (2026-07-24)

| Role | Org | P3 scored | P3 should be | Delta |
|------|-----|-----------|--------------|-------|
| Head of Studio Management | ARRISE (iGaming, 12,000 staff, 8 countries) | 2 | 4-5 | +2 to +3 |
| Modernization Tech Lead | Luxoft Serbia (IT consulting, DXC subsidiary) | 2 | 4-5 | +2 to +3 |

Both scored P3=2 ("minimal transferable value"). But:
- ARRISE is a 12,000-person multinational in a regulated industry — governance,
  compliance, multi-jurisdictional operations all transfer.
- Luxoft/DXC is a global IT consulting firm — enterprise architecture, client
  delivery, and transformation practices directly transfer.

### Prior calibration — PMI (2026-06-09, from scoring engine references)

The scoring engine's `pmi-private-sector-scoring-pattern-2026-06-09.md` established:
- PMI (global tobacco multinational): P3 = 7
- Private sector ceiling: 8
- Private sector base for large global companies: 5-7

## Corrected P3 table for private sector

| Private-sector org profile | P3 | Rationale |
|---|---|---|
| Large global multinational with transformation story (PMI, Luxoft/DXC, Accenture) | 6-8 | Global scale + digital transformation agenda |
| Mid-size regional company, regulated industry (ARRISE, local telecom) | 3-5 | Regulated ops transfer, no development mission |
| Startup / niche company, no development angle | 2-3 | Minimal transferable value |
| Shell company / fraudulent posting | 0-1 | Do not score |

**Rule: Do NOT score private-sector P3 as 0-1** for legitimate companies. Even
without a development mission, large private-sector employers have transferable
value: scale, governance, regulated industry experience, multi-jurisdictional
operations. The minimum for a legitimate mid-to-large company is 2; 0-1 is
reserved for shell companies or fraudulent postings.

## Impact on total scores

With corrected P3, the two roles from this session would score:

| Role | P3 was | P3 should be | Total was | Total should be | Verdict change |
|------|--------|-------------|-----------|----------------|----------------|
| ARRISE Head of Studio | 2 | 4 | 70 | 72 | No change (still APPLY SELECTIVELY) |
| Luxoft Modernization Tech Lead | 2 | 4 | 65 | 67 | No change (still APPLY SELECTIVELY) |

The 2-point delta doesn't change verdicts here, but it would matter for roles
scoring 73-74 where +2-3 points pushes them from APPLY SELECTIVELY to APPLY
(75+ threshold). Future sessions MUST use the corrected P3 table.

## Required scoring-engine SKILL.md patch

The `vaccancy-compatibility-scoring-engine` SKILL.md P3 table needs this row
added after "Other development orgs":

```
| Private sector (non-dev) | 2-8 | See references/private-sector-scoring-calibration-2026-07.md |
```

**Note:** The scoring engine skill is pinned. The patch was attempted but
rejected by the tool's pinned-skill protection. The user should run
`hermes curator unpin vaccancy-compatibility-scoring-engine` to allow the
SKILL.md patch, or apply the P3 table change manually.
