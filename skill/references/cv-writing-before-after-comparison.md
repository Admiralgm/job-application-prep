# Before / After: CV Style Transformation

## Summary

The AI-generated CV (v1) was 305 lines, ~15KB. The user manually edited it down to ~90 lines, ~5.5KB — a 3.4x compression. The user's version was adopted as the canonical format.

## Structural Changes

| Element | BEFORE (v1, AI) | AFTER (v2, User) |
|---------|-----------------|-------------------|
| Lines | ~305 | ~90 |
| Size | ~15KB | ~5.5KB |
| Separator lines | `====` headers between all sections | None |
| Summary | Full paragraph (8 lines) | 4 compact bullets |
| Competencies | 5 categories with sub-bullets (20+ lines) | 4 categories, inline • lists (4 lines) |
| Experience bullet count | 4-6 per role | 2-3 per role |
| Standalone case study | Yes (COVID Webex section, 15 lines) | No — folded into Summary bullet #2 |
| UNICEF roles | 2 separate entries | 1 merged entry |
| Certifications | Listed (6 lines) | Removed |
| Location section | Standalone (4 lines) | Folded into Summary opening paragraph |
| Additional experience | Mini-bullets per role (9+ lines) | One-liners (3 lines) |

## Bullet Density Comparison

### BEFORE (single-fact bullets, 3 lines):
```
• Built and operationalise AI Agent frameworks (Hermes, OpenClaw), including
  custom binary modification to bypass hardcoded safety budget timeouts
```

### AFTER (compound bullet, 1 line):
```
• AI Architecture & Deployment: Built and operationalized custom AI Agent frameworks with binary modifications, deployed local LLM workflows, and set up MCP server architectures.
```

The AFTER version packs the same information + additional facts (LLM workflows, MCP) into one line instead of three.

## What Was Cut Completely

- All `====` separator lines (~30 lines)
- Case Study standalone section (~15 lines) — content moved to Summary bullet
- Certifications section (~6 lines)
- Location & Eligibility standalone section (~4 lines) — content moved to Summary
- Redundant/verbose phrasing across all bullet points (~150 lines of fat)
- "END OF CV" marker

## What Changed in Wording

- "Markovic" → "Marković" (diacritic)
- Phone masked (****) → full number (+381 64 110 8335)
- "Serbian & EU (Czech Republic)" → "Dual National – Czech Republic (EU) & Serbia"
- "possesses a unique ability to act as a high-level coordination bridge" → "The Bridge: Combines the strategic vision... with the operational credibility..."
- Removed all filler: "extensive background in", "proven track record", "demonstrated expertise"
