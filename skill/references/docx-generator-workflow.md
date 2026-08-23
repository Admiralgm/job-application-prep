# DOCX CV Generator Workflow — Machine-Readable CV Production

> Captured from session 2026-07-25: 4 CVs generated for Tenstorrent, Zühlke, Rohlik, KPMG Malta.
> Generator script: `~/Desktop/LND/CVS/cv_generator.py`

## Overview

The DOCX CV generator (`cv_generator.py`) implements the §0.4 Reference-CV Formatting Lock from CV_REPOSITORY_DATABASE.md v26. It reads a JSON content file and produces a formatted `.docx` file with exact A4 geometry, Arial typography, 20pt blue name, 12pt headings, compound bullets, and footer.

## Generator Script Location

`~/Desktop/LND/CVS/cv_generator.py`

## JSON Content Directory

`~/Desktop/LND/CVS/` — both JSON input files and .docx output files live here.

## Workflow (per vacancy)

1. **Read the JD** — extract company, role, location, requirements, competencies
2. **Read the CV Repository** — `~/CV_REPOSITORY_DATABASE.md`
3. **Determine positioning lane** (from §C.1):
   - AI/Data/Digital Transformation → INCLUDE personal AI-stack material (Hermes, DGX-B200, token burn, MCP, LLM deployment)
   - C-suite/General Management → EXCLUDE personal AI-stack material; focus on COO, P&L, multi-country governance, M&A
   - Director/Head of Function → Include AI/DT but frame through transformation consulting lens
4. **Write JSON content file** to `~/Desktop/LND/CVS/cv_content_<Company>_<Role>.json`
5. **Run generator**: `python3 ~/Desktop/LND/CVS/cv_generator.py ~/Desktop/LND/CVS/cv_content_<Company>_<Role>.json`
6. **Verify**: `ls -la ~/Desktop/LND/CVS/User_CV_<Company>_<Role>.docx`

## JSON Content Structure

```json
{
    "headline": "TECHNICAL PROGRAM MANAGER — AI/ML",
    "profile": "150-word professional profile paragraph...",
    "competencies": [
        "Category: item1 • item2 • item3 • item4",
        ...
    ],
    "experience": [
        {
            "title": "ROLE TITLE (Contract type if Present)",
            "org": "Company Name",
            "location": "City, Country",
            "dates": "Month YYYY – Month YYYY/Present",
            "bullets": [
                "Compound bullet with label: fact 1, fact 2, and fact 3.",
                "Key Achievement: one crisp sentence."
            ]
        }
    ],
    "additional_experience": [
        "ROLE | Company | Location | Dates — One-liner compound sentence."
    ],
    "education": [
        "• Degree, Institution, Dates"
    ],
    "languages": "English – Fluent/Business | Serbian – Native | Russian – Fluent/Business",
    "company": "CompanyName",
    "role": "Role_Identifier"
}
```

## Positioning Lane Rules (from §C.1)

| Lane | AI-stack material | Lead experience | Focus |
|------|-------------------|-----------------|-------|
| AI/Data/Digital Transformation | INCLUDE (Hermes, DGX-B200, token burn, MCP, LLM) | Olivia Education | AI use cases, automation, agentic |
| C-suite/General Management | EXCLUDE entirely | HRAM COO | P&L, team scale, multi-country |
| Director/Head of Function | Include but frame as consulting | KPMG-style advisory | Portfolio, governance, transformation |
| Technical/Architecture | INCLUDE | Tetra Pak IT, Algotech | System design, integration |
| Telecom/ICT | INCLUDE | Globaltel, ZAMTEL, Uganda Telecom | Network, wholesale, MVNO |
| Sales/Business Development | EXCLUDE | Globaltel, Liquid Telecom | Pipeline, contracts, revenue |
| FinTech/Payments | EXCLUDE | Globaltel (BUSPLUS, mPARKING) | Transaction scale, compliance |

## Cross-Profile Notification

After generating all CVs, notify the requesting profile via cmux:
```bash
cmux tree --all --id-format uuids  # find target workspace UUID
cmux send --workspace <UUID> "AGENT CV GENERATION COMPLETE: N CVs created"
cmux send-key --workspace <UUID> Enter
```

## Pitfalls

- **Name must be User MARKOVIĆ** (with diacritic ć, Unicode U+0106) — the generator uses `\u0106` in the Python code
- **No first-person pronouns** in CV body — use action-led phrasing
- **No tables** for competencies — ATS parsers break on table cells
- **Compound bullets only** — pack 2-3 facts per bullet, never single-fact bullets
- **One Key Achievement per major role** — last bullet, one crisp sentence
- **Contract-type on Present roles** — if role started < 6 months ago, add `(Project-Based / Advisory)` or similar
- **Citizenship line**: "Belgrade, Serbia | EU citizen (Czech Republic)" — always in contact block
- **Max 3 A4 pages** — the generator enforces this via margins and spacing
- **Fresh tailoring per employer** — never copy JSON between CVs; rebuild each
