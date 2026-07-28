# Remote Vacancies Extraction — Portal Techniques & Scoring Corrections (2026-07-21)

## Context
Session ran the remote-vacancies-search workflow: scanned 280 Remotive jobs + Reed.co.uk searches (AI Director, Head of AI, Transformation Director). 8 candidates triaged, 6 added to tracker (IDs 41-46). This reference documents techniques and corrections that apply to any remote job scanning session.

---

## 1. Reed.co.uk — HIGHEST-YIELD PORTAL (Undervalued in remote-vacancies-search skill)

The remote-vacancies-search skill says Reed.co.uk returns "Search results with titles" only. This is WRONG — Reed returns full JDs via curl.

### Search URL Pattern
```
https://www.reed.co.uk/jobs/remote-{keyword}-jobs
```
Examples that worked:
- `remote-ai-director-jobs` → 25 results
- `remote-head-of-ai-jobs` → 25 results
- `remote-transformation-director-jobs` → 25 results

### Full JD Extraction (curl, no browser needed)
Reed embeds JD in JSON inside the page HTML:
```python
desc = re.search(r'"description":\s*"([^"]{200,})"', html)
text = desc.group(1).replace('\\n', '\n').replace('\\"', '"').replace('&amp;', '&')
text = re.sub(r'<[^>]+>', ' ', text)  # strip HTML tags
```
Returns COMPLETE job descriptions (2000-5500 chars typical).

### Yield
- 5 of 6 new tracker entries came from Reed.co.uk
- Key roles: CAIO (£325-350k), AI Transformation Director (£1200/day), Head of Labs AI/LLMs, Group IT Director
- Reed.co.uk is a FIRST-TIER portal for UK/remote senior roles — scan early, not as an afterthought

---

## 2. Remotive — Search Page JSON (Not Just RSS)

The remote-vacancies-search skill says RSS feed returns "34 most recent jobs." The search page contains `window.__INITIAL_SEARCH_RESULTS__` with 280+ jobs in JSON.

### Extraction
```python
html = fetch_url("https://remotive.com/remote-jobs/search?category=artificial-intelligence")
match = re.search(r'window\.__INITIAL_SEARCH_RESULTS__\s*=\s*({.*?});\s*</script>', html, re.S)
data = json.loads(match.group(1))
# Jobs in data["jobs"] — each has "title", "company_name", "url", "location", "salary", "description"
```
280 jobs with full metadata vs 30 from RSS. Use as primary Remotive extraction method.

---

## 3. ATS Board Extraction Patterns

When a listing links to an external ATS (Applicant Tracking System), the ATS domain determines extraction method:

| ATS Domain | Method | Notes |
|------------|--------|-------|
| `greenhouse.io` | curl (no browser) | Full JD in HTML body, 8000+ chars. Tested: Xebia, Dragos |
| `bamboohr.com` | browser + 5s delay | JS-rendered. `browser_console(expression="document.body.innerText.substring(0, 6000)")`. Tested: GoCharting (worked), ICS.AI (failed) |
| `ashbyhq.com` | try browser, expect expiry | Many postings return "Job not found". Tested: SoSafe, deepset — both expired |

### Pattern: Check ATS Before Fetching
When a listing links externally, identify the ATS by URL domain:
- `greenhouse.io` → curl works, fetch directly
- `bamboohr.com` → needs browser, add 5s delay
- `ashbyhq.com` → try browser, expect "Job not found" for expired
- Unknown ATS → try curl first, fall back to browser

---

## 4. Post-Brexit P6/P3 Logistics Scoring Correction

The remote-vacancies-search skill says "UK remote → 10 (Czech citizenship)." This is WRONG post-Brexit.

### Correct Scoring for UK Roles
- Czech EU citizenship does NOT confer UK work rights post-Brexit
- UK roles without explicit sponsorship → P6 = 6-7 (visa uncertainty)
- UK roles WITH "no visa sponsorship" clause → HARD_NO_WORK_AUTHORIZATION_SPONSORSHIP (score 0)
- UK/EMEA remote (not UK-specific) → P6 = 9 (EMEA scope)

### This Session's Impact
| Role | P6 Score | Reasoning |
|------|----------|-----------|
| Enterprise Account Director (UK) | 0 (HARD NO) | "Visa sponsorship is not available" |
| AI Transformation Director (UK Hybrid) | 7 | No sponsorship clause, but post-Brexit uncertainty |
| Head of Labs (UK Remote) | 7 | Same reasoning |
| Group IT Director (UK) | 6 | National remit, travel required, more uncertainty |
| CAIO (UK/EMEA Remote) | 9 | EMEA scope, not UK-specific |

### Updated P6 Table for UK
| Condition | P6 Score | Notes |
|-----------|----------|-------|
| UK + "no sponsorship" clause | 0 (HARD NO) | Disqualify entirely |
| UK + no sponsorship clause | 6-7 | Post-Brexit EU citizen uncertainty |
| UK/EMEA remote (not UK-specific) | 9 | EMEA scope, not UK-only |
| UK + confirmed sponsorship | 8 | Rare but possible |

---

## 5. Duplicate Check Method

Before adding new entries, check against existing tracker:
```python
# Read tracker, extract existing titles
# Compare new candidates against existing using word-overlap similarity
# Threshold: >50% word overlap = flag for manual review
# Note: overlap flags are often false positives (different companies, different roles)
# Always manually verify flagged "duplicates" before excluding
```

This session: "Senior / Principal Applied AI Engineer" flagged vs "Senior Independent AI Engineer / Architect" — false positive, different companies and roles.

---

## 6. Tracker Update Process

1. Insert new summary entries before the separator line in the summary table
2. Append detailed entries at the end of the file
3. Update header stats (total scanned, deep-scored, new this session)
4. Run `sync` after writing
5. Verify by reading back key sections (summary entries, detailed entries, disqualified section)
