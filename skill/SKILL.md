---
name: job-application-prep
description: "Evaluate job vacancies against User's profile, produce tailored CVs and cover letters, and answer application form questions. Uses the canonical compact CV style (max 3 A4 pages, compound bullets, zero fluff). Includes the full CV-writing playbook: tailoring rules, wrong-vs-right comparisons, forbidden elements checklist, and the COVID Webex / 'Bridge' differentiator patterns."
---

# Job Application Preparation

> **Purpose:** End-to-end workflow: evaluate a job posting → decide APPLY/SKIP → produce tailored CV, cover letter, and application form answers. All materials follow User's canonical compact style.

## When to Use

- User pastes a job description and asks "evaluate this," "should I apply," or "profile compatibility"
- User asks for a tailored CV or cover letter for a specific role
- User needs answers to application form questions (e.g., "describe your experience in X")
- User compares CV versions or asks for CV analysis
- User asks you to fill in an online application form in their browser (work experience, skills, education sections) — see Phase 5

## Phase 1: Vacancy Evaluation

Use the scoring framework from the wiki. Load these pages for reference:
- `config/wiki/concepts/profile-master-summary.md` — complete skills inventory
- `config/wiki/concepts/vacancy-compatibility-evaluation-system.md` — scoring methodology
- `config/wiki/entities/User-markovic.md` — master profile

**Output:** A compatibility report with score (0-100), domain/geographic/role/seniority/keyword breakdowns, red flags, enablers, and a clear APPLY or DO NOT APPLY verdict.

**Decision thresholds:**
- >= 85: STRONG FIT — apply immediately
- 70-84: COMPETITIVE — apply within 48h
- 55-69: STRETCH — apply only if strategic
- 40-54: LONG SHOT — low priority
- < 40: SKIP

**Hard blockers (always SKIP):**
- Language requirement User doesn't meet (German C1, French mandatory, etc.)
- National Officer (NO-) grades
- Ukraine-based
- "Nationals only" without EU exception
- Requires specific certification User doesn't have (verify if genuinely mandatory)

**Always note these in the report:**
- Critical gaps that disqualify
- The "alternative path" door if the JD has one (User often qualifies through "technical degree + deliberate pivot into business" clauses)
- What User brings that competitors likely don't (EU + Russian + AI hands-on is a rare trifecta)

### JD Source: LinkedIn Jobs Tracker (NEW — 2026-07-24)

When the user asks to evaluate jobs saved on LinkedIn's Jobs Tracker
(`https://www.linkedin.com/jobs-tracker/?stage=saved`), use AppleScript +
Chrome `execute javascript` to extract the full JDs from the user's
authenticated session. The tracker page shows job cards with title,
company, and location — but NOT the full JD. Each card links to the full
posting at `linkedin.com/jobs/view/JOB_ID/`.

**Technique:** Write JS to `/tmp/*.js`, wrap in AppleScript `.scpt`, run
via `osascript`. Navigate to each job posting URL, wait 5s, extract JD
from `document.querySelector('main').innerText`. See
`references/linkedin-jobs-tracker-extraction-2026-07-24.md` for the full
step-by-step with code samples.

**Note:** "Requirements added by the job poster" at the bottom of LinkedIn
JDs are hard requirements — treat them as mandatory screening criteria,
not desirables.

### JD Source: LinkedIn Search Results (NEW — 2026-07-24, updated 2026-07-25)

When the user pastes a LinkedIn job **search** URL (e.g.
`linkedin.com/jobs/search/?...&geoId=...&savedSearchId=...`), extract
job IDs from the search page, then fetch full JDs via a multi-method
fallback chain:

1. **Extract job list** from the search page via AppleScript+Chrome
   (`document.querySelectorAll('a[href*="/jobs/view/"]')`). Each page
   shows ~8-10 jobs. Paginate with `&start=25` for page 2.

2. **Supplement with guest API ID discovery** — The endpoint
   `seeMoreJobPostings/search?keywords=...&start=0` returns ~30KB HTML
   with `data-job-id` attributes. This finds 10+ additional IDs beyond
   what Chrome renders. Only `start=0` works (pagination returns empty).
   See `references/linkedin-search-extraction-curl-cookie-2026-07-24.md`
   for the full technique.

3. **Fetch full JDs** — AppleScript Chrome navigation works for the first
   ~3 jobs, then LinkedIn throttles dynamic content rendering. Switch to
   **curl + Chrome cookies** for all remaining jobs:
   - Extract `document.cookie` from Chrome via AppleScript
   - Write cookies in Netscape format to `/tmp/cookies.txt`
   - `curl -s --max-time 10 -b /tmp/cookies.txt "https://www.linkedin.com/jobs-guest/jobs/api/jobPosting/{JOB_ID}"` returns ~66KB HTML per job
   - Strip HTML tags with Python regex — the JD starts after LinkedIn's sign-in boilerplate

   This method reliably fetches ALL JDs with zero throttling at 1s
   intervals. See
   `references/linkedin-search-extraction-curl-cookie-2026-07-24.md`
   for the full technique with code samples.

**⚠️ URL format pitfall:** The user may provide a `/jobs/search-results/`
URL (generated from LinkedIn in-app notifications). This format renders
only 1 job card in Chrome (the currentJobId). Navigate Chrome to it to
establish the session, then switch to `/jobs/search/` format for
extraction, or use the guest API for ID discovery. The search-results
format is less useful for extraction.

**⚠️ What doesn't work:** Scrolling the search page does NOT load more
jobs (pagination, not infinite scroll). Clicking job cards in split view
returns empty JD content after throttling. Inlining cookies in curl
`-H "Cookie: ..."` breaks on special characters — always use
`-b /tmp/cookies.txt`. The Voyager API returns 0 results.

### JD Source: Email Job Alerts (NEW — 2026-08-02)

When the user asks to retrieve jobs from email alerts (LinkedIn Job
Alerts, EY/SuccessFactors job alerts, or any `*@noreply*.jobs2web.com`
sender), use himalaya to search, read, and parse job URLs from email
bodies, then fetch full JDs and score them.

**Workflow:**

1. **Search by sender** — use `himalaya --output json envelope list --page 1
   --page-size 50 from "sender@domain"` to find alert emails. Parse JSON
   with Python for structured output (ID, subject, date).

2. **Read email body** — `himalaya message read <ID>` returns plain text
   including HTML part content. Job alerts contain job titles + URLs
   separated by `\r\n` and dashes.

3. **Parse job URLs** — extract job IDs from URLs in the email body.
   LinkedIn URLs contain `/jobs/view/<JOB_ID>/`. EY/SuccessFactors URLs
   contain `/job/<Location>-<Title>/<JOB_ID>/`.

4. **Fetch full JDs** — use the right tool per portal:
   - **LinkedIn guest API** — `curl -sL -o /tmp/job.html "https://www.linkedin.com/jobs-guest/jobs/api/jobPosting/<JOB_ID>"` then strip HTML with Python regex. No cookies needed. Returns ~66KB HTML per job. Works for active postings; returns empty for expired ones.
   - **EY / SuccessFactors** — `curl` returns only JS boilerplate (GTM scripts, cookie consent). MUST use `browser_navigate` to load the page — the browser renders the JS and the accessibility snapshot contains the full JD text.
   - **Other jobs2web.com portals** (ILO, UNIDO, ICRC, ITU, etc.) — try curl first; if the response is JS boilerplate, fall back to `browser_navigate`.

5. **Filter for relevance** — before fetching JDs, scan job titles in the
   email and skip obvious mismatches (actuarial, insurance, mechanical
   engineering with no ICT content, etc.). Fetch only relevant JDs.

6. **Score with V5.0.0** — load `vacancy-compatibility-scoring-engine`
   and `cv-repository` skills, read the CV database, then score each JD
   with full 7-parameter framework.

**LinkedIn sender address distinction (important):**
- `jobalerts-noreply@linkedin.com` — sends **job alert digests** (multiple jobs per email, subject = top job title). This is the one to search for.
- `jobs-noreply@linkedin.com` — sends **application confirmations** and single-job recommendations. Not a job alert source.

**Himalaya filter limitation:** Cannot combine `subject "X" from "Y"` in
one query — the parser rejects it with "found 'f' expected space between
filters". Use `from "sender"` alone, or `subject "keyword"` alone. Never
combine filters.

See `references/email-job-alert-extraction-2026-08-02.md` for the full
worked example with code samples, sender addresses, and portal-specific
fetch techniques.

## Phase 2a: DOCX CV Generation Pipeline (JSON → Generator → .docx)

When producing machine-readable .docx CVs via the generator script at `~/Desktop/LND/CVS/cv_generator.py`:

### Positioning Lane Rules (from §C.1 — determines AI-stack material inclusion)

| Lane | AI-stack material (Hermes, DGX-B200, token burn) | Lead experience | Focus |
|------|---------------------------------------------------|-----------------|-------|
| AI/Data/Digital Transformation | **INCLUDE** | Olivia Education | AI use cases, automation, agentic |
| C-suite/General Management | **EXCLUDE entirely** | HRAM COO | P&L, team scale, multi-country |
| Director/Head of Function | Include but frame as consulting | KPMG-style advisory | Portfolio, governance, transformation |
| Technical/Architecture | INCLUDE | Tetra Pak IT, Algotech | System design, integration |
| Telecom/ICT | INCLUDE | Globaltel, ZAMTEL, Uganda Telecom | Network, wholesale, MVNO |
| Sales/Business Development | EXCLUDE | Globaltel, Liquid Telecom | Pipeline, contracts, revenue |
| FinTech/Payments | EXCLUDE | Globaltel (BUSPLUS, mPARKING) | Transaction scale, compliance |

### Workflow per vacancy

1. **Read the JD** — extract company, role, location, requirements, competencies
2. **Read the CV Repository** — `~/CV_REPOSITORY_DATABASE.md`
3. **Determine positioning lane** from table above
4. **Write JSON content file** to `~/Desktop/LND/CVS/cv_content_<Company>_<Role>.json`
5. **Run generator**: `python3 ~/Desktop/LND/CVS/cv_generator.py ~/Desktop/LND/CVS/cv_content_<Company>_<Role>.json`
6. **Verify**: `ls -la ~/Desktop/LND/CVS/User_CV_<Company>_<Role>.docx`

### JSON Content Structure

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

### Cross-Profile Notification

After generating all CVs, notify the requesting profile via cmux:
```bash
cmux tree --all --id-format uuids  # find target workspace UUID
cmux send --workspace <UUID> "AGENT CV GENERATION COMPLETE: N CVs created"
cmux send-key --workspace <UUID> Enter
```

### Pitfalls
- **Name must be GORAN MARKOVIĆ** (with diacritic ć, Unicode U+0106) — the generator uses `\u0106`
- **No first-person pronouns** in CV body — use action-led phrasing
- **No tables** for competencies — ATS parsers break on table cells
- **Compound bullets only** — pack 2-3 facts per bullet, never single-fact bullets
- **One Key Achievement per major role** — last bullet, one crisp sentence
- **Contract-type on Present roles** — if role started < 6 months ago, add `(Project-Based / Advisory)` or similar
- **Citizenship line**: "Belgrade, Serbia | EU citizen (Czech Republic)" — always in contact block
- **Max 3 A4 pages** — the generator enforces this via margins and spacing
- **Fresh tailoring per employer** — never copy JSON between CVs; rebuild each

## Phase 2b: CV Creation — Canonical Compact Style

> **GOLDEN RULE:** The user HATES verbose CVs. Version 1 of the refurbed CV was 305 lines — the user cut it to 61. Every future CV must start from that compact standard, not from my natural verbose tendency.
>
> **FOUNDATIONAL STEP BEFORE WRITING:** Update the CV database with User's current Present-role work before generating the CV. In the WTO session (Jun 2026, ref: current-work-overrides-database-2026-06), User's active Moodle/Canvas LMS integration at Olivia Education was NOT in the database — it had to be added mid-session via patch on section 3.1. Check and update section 3.1 first: if User's current work overlaps the target role's core function, patch the database BEFORE generating materials. This prevents stale-sourced CVs that miss the candidate's strongest current proof points.
>
> **MASTER REPOSITORY:** `~/Desktop/CV/User_CV_Repository.docx` — modular system: 4 summary options, 4 competency clusters, 12 experience entries with full bullet text, 3 distinctive projects, 3 ATS keyword banks. Build CVs by selecting components from the repository, then apply canonical formatting. See `cv-writing` skill references for inventory guide.
>
> **HUMANIZER:** ALL user-facing output (CVs, cover letters, form answers, evaluations) must be humanized. Load the `humanizer` skill and apply it to every piece of prose. No AI-slop vocabulary, no formal discourse markers, no perfectly symmetrical paragraphs, no generic conclusions. Write like a person who did the work.

### Structural Rules (IRON — never violate)

| Rule | Detail |
|------|--------|
| **Max length** | 3 A4 pages. If it doesn't fit, cut harder. |
| **No fluff** | No "extensive experience in," "proven track record of," "I am a seasoned..." — delete all filler. Every word earns its place. |
| **Bullets** | Compound sentences with bold category labels: `**Category:** fact 1, fact 2, and fact 3.` Pack multiple facts into one bullet. |
| **Key Achievement** | Each major role ends with ONE "Key Achievement:" bullet — a single crisp sentence. Not two, not a paragraph. |
| **Additional experience** | Old/less-relevant roles go here as one-liners: `ROLE \| Company \| Dates — Single compound sentence.` No sub-bullets. Max 3 entries. |
| **Merge** | Adjacent similar roles merge into one entry (UNICEF GIGA + Learning Passport → single block). |
| **No separators** | No `====`, `---`, or "END OF CV" markers. Section titles in ALL CAPS only. |
| **No standalone sections** | No "Case Study" section (fold into a bullet). No "Certifications" section (unless the role explicitly requires certs). No "Location & Eligibility" section (fold into Summary). |

### Canonical Structure (in order)

```
1. HEADER
   GORAN MARKOVIĆ (with ć, not c)
   Belgrade, Serbia | +381 64 110 8335 | admiralGM@gmail.com
   LinkedIn: linkedin.com/in/goran-markovic3229
   Citizenship: Dual National – Czech Republic (EU) & Serbia

2. PROFESSIONAL SUMMARY
   - Opening paragraph (2-3 sentences): identity, years, geography, EU status
   - 4 bulleted highlights, each with bold category label
   - Bullet 4 should be the "bridge" — strategic + operational combo

3. CORE COMPETENCIES
   - 4-6 categories, each as: "**Category:** item1 • item2 • item3 • item4"
   - Inline • separators only — no sub-bullets, no line breaks between items
   - If the role mentions specific vendors/tools, name-check them here

4. PROFESSIONAL EXPERIENCE
   - 3-5 roles, reverse chronological
   - Each: ROLE TITLE, Organization | Location | Dates
   - **⚠️ Contract-type rule:** Any role that started within the last 6 months and is listed as "Present" MUST include its contract type in the title: `(Project-Based / Advisory)`, `(Contract)`, `(Interim)`. Prevents "job hopper" inference.
   - 2-3 compound bullets per role (3 max for the most important role)
   - Last bullet per role: "Key Achievement: ..."
   - Reorder to put the most role-relevant experience FIRST

5. ADDITIONAL RELEVANT EXPERIENCE
   - One-liners only, no sub-bullets, max 3 entries

6. EDUCATION (bottom, compact)

7. LANGUAGES (bottom, inline format)
   English – Fluent | Russian – Fluent | Serbian – Native

8. LOCATION & RELOCATION (only if relocation is a decision factor)
   One line, factual, low-friction framing.
```

### Tailoring Rules: The Role-to-CV Map

**The Bridge** — Paragraph 4 of the Professional Summary should always show that User is BOTH strategic AND operational. Concrete example: User reached out to a stranger on LinkedIn during COVID (Webex CEO's Chief of Staff), came prepared with a solution (Cisco Webex used in Serbia for patient-family communication in COVID isolation), and delivered a solution within 48 hours that ended up being used by hospitals. This one story ("The Bridge") proves strategic initiative (reaching out cold to a C-suite executive) AND operational execution (working through the night to ship a working prototype). It's the single strongest differentiator in User's career. Feature it.

**Role-first reordering:** Always put the most role-relevant experience first — even if it's not the most recent. The reader decides in 6 seconds.

**Explicitly forbidden elements (never include):**
- ❌ Filler: "extensive experience", "proven track record", "seasoned professional"
- ❌ Separator lines (`====`, `---`, visual dividers between sections)
- ❌ Standalone "Case Study" sections (fold into a bullet)
- ❌ Standalone "Certifications" section (unless JD explicitly requires certs)
- ❌ Listing every technology ever touched — only name-check what this role needs
- ❌ Simple list formatting — every bullet should be a compound sentence

**Citizenship line:** Always include "Dual National – Czech Republic (EU) & Serbia" — this is the single most important line for remote/European roles.

### Comparison: What vs. What Not

A wrong CV bullet reads like a job description:
> "Responsible for managing cross-functional teams and driving strategic initiatives across multiple geographies."

The right version reads like proof:
> **Cross-functional leadership:** Led 15+ person engineering, product, and DevRel teams across 3 countries (Serbia, Zimbabwe, Cyprus) delivering UNICEF's GIGA connectivity program to 1,000+ schools.

Same content density, zero fluff. If a bullet could appear on anyone's CV, delete it.

## Pitfalls (things I've done wrong and been corrected on)

## Phase 3: Cover Letter Creation

**Style rules:**
- SHORT. One screen max. The user asked for "a short cover letter" — respect that.
- No preamble ("I am writing to express my strong interest in..." — NEVER)
- Opening: direct, contextual, specific to the role. Quote their JD if there's an "alternative path" clause.
- Each paragraph: one requirement from the JD → one proof from User's experience with a metric.
- Closing: confident, brief. "I would welcome the opportunity to discuss..."
- Include phone and email in the signature.

**Structure:**
```
Subject: [Role Title] — Goran Marković

Dear [company] team,

[Opening — 2-3 sentences. Hook them with specific relevance.]

[Paragraph 2 — primary strength matching their #1 requirement. Metric-backed.]

[Paragraph 3 — secondary strength. Proof point with numbers.]

[Paragraph 4 (optional) — address a gap or add a differentiator.]

[Closing logistics — EU citizen, location, availability. 1-2 sentences.]

[Confident sign-off — 1 sentence.]

Best regards,
Goran Marković
phone
email
```

**Cover letter patterns for common org types:**
- UN/UNICEF: Lead with UN insider status, GIGA/Learning Passport, education anecdotes
- Private sector tech: Lead with hands-on AI + operational scale
- FinTech: Lead with payment systems (BUSPLUS 500K tx/day) + compliance
- Consulting firms: Lead with multi-country delivery + stakeholder management

Reference: `config/wiki/concepts/cover-letter-pattern-library.md`

## Phase 4: Application Form Answers

When the user pastes a form question (e.g., "Please describe your experience in change-management."):

- **Direct opening.** "My change-management experience follows a consistent pattern..." — not "I have extensive experience in..."
- **One clear proof point per paragraph.** A specific story with numbers, not a list of adjectives.
- **Honest about gaps.** If the question asks "Have you led X?" and the answer is "not quite," say: "My answer is honest: I have done Y but not X — and that is precisely the step I am ready to take here." This builds trust.
- **Close strong.** End with a pattern/principle, not a weak "I hope..."
- **Keep it under 250 words** unless the form demands more.
- **Be honest about gaps.** When a question asks "Have you led X?" and the answer is "not quite," the winning pattern is: "My answer is honest: I have done Y but not X — and that is precisely the step I am ready to take here." This builds trust. Never pad or overclaim — recruiters smell it.
- **Always label recent roles with contract type.** Any role that started within the last 6 months and is listed as "Present" MUST include `(Project-Based / Advisory)`, `(Contract)`, or `(Interim)` in the title line. This prevents the "job hopper" inference. Learned from Claude Opus 4.5 review — flagged Olivia Education (March 2026–Present, 2 months in) as a risk without this clarification.

## Pitfalls (things I've done wrong and been corrected on)

1. **Verbose CVs.** My natural tendency is to write long, comprehensive CVs with standalone case study sections, certification lists, and detailed bullet points. The user cut my 305-line CV to 61 lines. The canonical style is LEAN. When in doubt, cut more.

2. **Separator lines.** I used `====` and `---` extensively. The user removed them all. Section titles in ALL CAPS are sufficient.

3. **Standalone sections for everything.** I created separate sections for Case Studies, Certifications, Location & Eligibility. The user folded all of these into bullets or the Summary. Only create a standalone section if the role explicitly requires it and NOT including it would hurt.

4. **Too many bullets per role.** I wrote 4-6 bullets per role. The user cut to 2-3 max. One Key Achievement at the end.

5. **Not aggressive enough on tailoring.** For the Reluna role, Gemini 3.1 Pro's CV mirrored the JD language more aggressively (named their vendors, used phrases like "operational bridge" and "removing blockers" that matched the JD). The lesson: read the JD for exact language and echo it.

6. **Folding too many roles into "Additional Experience."** Dropping Algotech entirely was a mistake — keep all significant roles as at least one-liners.

7. **Missing contract-type on recent roles (NEW — 2026-05-24).** Claude Opus 4.5 flagged the Olivia Education role (March 2026–Present, 2 months in) as a risk — why is this person applying elsewhere so soon? Always label roles that started < 6 months ago with their contract type: `(Project-Based / Advisory)`, `(Contract)`, `(Interim)`. This kills the "job hopper" inference before it forms.

8. **Weak location framing (NEW — 2026-05-24).** Never write "Based in Belgrade, ready to relocate." Reframe geography as strategy: "Uniquely positioned to manage Reluna's Serbian jurisdiction on the ground while executing a high-frequency hybrid presence in Nicosia, Cyprus." Gemini 3.1 Pro taught this — location is a competitive advantage, not a logistical note.

9. **Form answers need honesty about gaps (NEW — 2026-05-24).** When asked "Have you led X?" and the answer is "not quite," the winning pattern is: "My answer is honest: I have done Y but not X — and that is precisely the step I am ready to take here." This builds trust. For the Reluna "led internal AI/engineering teams" question, the honest frame was: led classical IT at scale, AI leadership has been hands-on/advisory, and that's exactly the step I'm ready to take. This pattern generalizes to all edge-case form questions.

10. **Missing diacritic in name (NEW — 2026-05-24).** The header must always be "Goran Marković" with ć, not "Goran Markovic" with c. Using the Latin 'c' instead of the correct Serbian 'ć' is the kind of detail that signals carelessness on a document that supposedly represents you. This was caught by Claude Opus 4.5 review.

11. **Over-separating sections (NEW — 2026-05-24).** Resist the urge to add visual separators, horizontal rules, or extra whitespace between sections. All-caps section titles are sufficient. Visual clutter reduces content density and signals amateur formatting.

12. **Stale CV database (NEW — 2026-06-02).** The Olivia Education section 3.1 had been written before User started Moodle/Canvas LMS integration work. When the WTO Digital Learning Tech Specialist role came up, the initial score of 58/100 was based on a database that omitted this critical current work. The score jumped 14 points to 72/100 once §3.1 was patched with the current work. **Lesson:** Always ask User if his current role has evolved since the database was last updated, and patch §3.1 before writing CV materials.

13. **AppleScript "missing value" when filling browser forms (NEW — 2026-07-18).** When driving Chrome via AppleScript `execute javascript`, complex JS with `try/catch` blocks or bare `.click()` statements that return undefined produce the literal string "missing value" instead of a usable return value. Always end JS with an explicit string expression (e.g., `'result=' + field.value`). Never wrap in try/catch. If a click+return sequence fails, split into two separate `execute javascript` calls with `sleep 1` between them.

14. **Do NOT submit forms on the user's behalf (NEW — 2026-07-18).** When filling application forms in the user's browser, fill all fields visibly and leave the Submit button untouched. The user reviews the filled form in their own tab and clicks Submit themselves. This is the correct default — the user said "I want to monitor the whole process so do it visibly."

16. **LinkedIn search-results URL format (NEW — 2026-07-25).** The user provided a `/jobs/search-results/` URL (generated from LinkedIn in-app notifications). I used it directly and only got 1 job card. The user corrected me: the `/jobs/search/` format is the one that renders multiple job cards. **Lesson:** When the user provides a search-results URL, navigate Chrome to it to establish the session, then switch to `/jobs/search/` format for extraction, or use the guest API `seeMoreJobPostings/search?start=0` for ID discovery. The search-results format is less useful for extraction.

15. **Private-sector P3 scoring undercounts (NEW — 2026-07-24).** The V5.0.0 scoring engine P3 table has no row for pure private-sector roles. Without guidance, the scorer defaults to P3=0-2, which is systematically 3-6 points too low. The engine's own PMI calibration (2026-06-09) established private-sector P3 at 5-7 for large multinationals. This session scored ARRISE (12,000 staff, 8 countries) and Luxoft/DXC (global IT consulting) both at P3=2 — they should be 4-5. **Rule:** For legitimate mid-to-large private-sector companies, P3 minimum is 2; use 3-5 for mid-size regulated, 6-8 for large global multinationals. See `references/private-sector-scoring-calibration-2026-07.md` for the full corrected table and worked examples. **Action needed:** The scoring engine SKILL.md is pinned — user should run `hermes curator unpin vacancy-compatibility-scoring-engine` to allow the P3 table patch.

## Wiki Reference Pages

Load these from `config/wiki/` when preparing applications:

| Page | Use |
|------|-----|
| `concepts/profile-master-summary.md` | Complete skills inventory — the lossless reference |
| `concepts/vacancy-compatibility-evaluation-system.md` | Scoring methodology and decision thresholds |
| `entities/User-markovic.md` | Master profile with full career data |
| `concepts/cv-style-guide-canonical.md` | The canonical CV format rules |
| `concepts/cv-role-type-templates.md` | Legacy CV templates per role type |
| `concepts/cover-letter-pattern-library.md` | Cover letter patterns, pricing, and org-specific frames |
| `concepts/key-achievements-summary.md` | All quantifiable metrics |
| `raw/articles/User-full-cv-2026-05-23.md` | Full CV source text |

## Multi-Model CV Synthesis

When producing a CV for a high-stakes role, the recommended approach is three-model triangulation:
1. **Hermes baseline** — structure, accuracy, quantitative density
2. **Gemini JD-mirroring** — strategic framing, vendor naming, aggressive tailoring
3. **Claude risk review** — contract-type gaps, honest location framing, structural critique

See `references/reluna-multi-model-comparison.md` for a full worked example.

## References

All reference files are in `references/`:

| File | Content |
|------|---------|
| `cv-writing-before-after-comparison.md` | Before/after diff showing the transformation from verbose to compact CV |
| `cv-writing-canonical-template.txt` | The canonical CV template text |
| `cv-writing-repository-guide.md` | Full inventory of the modular CV repository |
| `cv-writing-reluna-merge-example.md` | Multi-model synthesis example from the Reluna application |
| `reluna-multi-model-comparison.md` | Full three-model comparison (Hermes/Gemini/Claude) |
| `humanizer-mandate.md` | Humanizer rules for all user-facing output |
| `vacancy-compatibility-scoring.md` | Seven-parameter job compatibility scoring engine |
| `unicef-detail-extraction.md` | UNICEF job detail extraction workflow |
| `remote-vacancies-extraction-2026-07-21.md` | Remote portal extraction patterns (Reed.co.uk full JDs, Remotive search page JSON, ATS board patterns, post-Brexit P6 UK scoring correction) |
| `linkedin-jobs-tracker-extraction-2026-07-24.md` | LinkedIn Jobs Tracker extraction via AppleScript+Chrome — scan saved jobs, navigate to each posting, extract full JD for scoring |
| `linkedin-search-extraction-curl-cookie-2026-07-24.md` | LinkedIn search results JD extraction — curl + Chrome cookies fallback chain. AppleScript throttles after ~3 jobs; curl with Netscape-format cookies reliably fetches all JDs via the guest API endpoint |
| `private-sector-scoring-calibration-2026-07.md` | Private-sector P3 scoring calibration — corrected P3 table for non-development roles (ARRISE, Luxoft, PMI pattern). Scoring engine SKILL.md is pinned; this reference holds the fix until unpinned |
| `email-job-alert-extraction-2026-08-02.md` | Email job alert extraction workflow — LinkedIn sender distinction, EY/SuccessFactors browser requirement, himalaya filter limitation, curl vs browser per portal |

## Phase 5: Automated Form Filling (NEW — 2026-07-18)

When the user asks you to fill in an online application form (work experience,
education, skills) in their **actual browser** — not a separate Hermes browser:

1. **Use AppleScript via terminal** to control Chrome (not browser_* tools, which
   open a separate unauthenticated session). See `macos-computer-use` skill
   reference: `references/applescript-chrome-form-fill.md` for the full technique.

2. **Source data from the CV Repository Database**
   (`~/Desktop/CV/CV_REPOSITORY_DATABASE.md`) — Section 3 has all 12 positions
   with periods, companies, and roles. Enter reverse-chronological.

3. **Fill visibly — do NOT submit.** The user monitors the process in their own
   browser tab. Leave the Submit button for them to click after review.

4. **Cookie banners first.** Dismiss any cookie consent overlay before
   interacting with form fields.

5. **React forms need the native value setter.** Direct `.value =` assignment
   silently fails. Use `Object.getOwnPropertyDescriptor(HTMLInputElement.prototype,
   'value').set`. React datepickers need focus → year dropdown select → month
   button click.

6. **AppleScript "missing value" return.** When `execute javascript` returns the
   literal string "missing value", the JS either returned undefined or a
   try/catch block disrupted the return path. Always end JS with an explicit
   string expression. Never wrap in try/catch.

7. **File-per-action pattern.** Write each AppleScript to `/tmp/*.scpt` via
   `write_file`, then execute with `osascript /tmp/*.scpt`. Inline
   `osascript -e` breaks on complex JS with quotes/newlines.

8. **Skip entrepreneurial/personal projects** (Alfa Net, football referee)
   unless the user explicitly asks — these aren't in Section 3 Professional
   Experience and can raise questions about overlapping employment.

### DATA REPOSITORY

All output files (tracker files, job descriptions, scan reports, CVs, cover letters, research outputs) MUST be written to:
`~/Downloads/DATA_REPOSITORY/`

Tracker files:
- `UN_SECTOR_VACCANCIES.txt` — Active reliable-source UN vacancies
- `UN_SECTOR_VACCANCIES_IMPACTPOOL.txt` — Active unreliable-source vacancies
- `UN_SECTOR_VACCANCIES_ARCHIVE.txt` — Applied/expired
- `FREELANCE_CONSULTING_OPPORTUNITIES.TXT` — Freelance/consulting
- `UN_SECTOR_VACCANCIES_TEMPLATE.txt` — Format reference
