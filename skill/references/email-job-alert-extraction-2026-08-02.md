# Email Job Alert Extraction — 2026-08-02

Worked example: retrieving jobs from LinkedIn Job Alerts and EY/SuccessFactors
job alert emails, fetching full JDs, and scoring them.

## Email Senders Identified

| Sender | Type | Content |
|--------|------|---------|
| `jobalerts-noreply@linkedin.com` | LinkedIn job alert digest | Multiple jobs per email (5-6), subject = top job title |
| `jobs-noreply@linkedin.com` | LinkedIn application confirmation | Single job applied + 3 recommended similar jobs |
| `EYJobAlerts@noreply12.jobs2web.com` | EY/SuccessFactors job alert | 10 jobs per email, HTML part with links |
| `*-jobnotification@noreply*.jobs2web.com` | UN/IGO job alerts | Similar pattern (ILO, UNIDO, ICRC, ITU, UNESCO, etc.) |

## Step 1: Search Email by Sender

```bash
# LinkedIn job alerts (correct sender)
himalaya --output json envelope list --page 1 --page-size 50 from "jobalerts-noreply@linkedin.com" | \
  python3 -c "
import sys,json
msgs=json.load(sys.stdin)
for m in msgs:
    frm=m.get('from',{})
    print(f'{m[\"id\"]} | {frm.get(\"name\",\"?\")} | {frm.get(\"addr\",\"?\")[:40]} | {m.get(\"subject\",\"?\")[:70]} | {m.get(\"date\",\"?\")[:16]}')
"
```

**Pitfall:** Cannot combine filters — `himalaya envelope list subject "jobs" from "linkedin"` fails with parser error. Use one filter at a time.

**Pitfall:** Searching `from "jobs-noreply@linkedin.com"` finds application confirmations, NOT job alerts. The alert sender is `jobalerts-noreply@linkedin.com` (note: `jobalerts-` not `jobs-`).

## Step 2: Read Email Body

```bash
himalaya message read <ID>
```

Returns plain text including HTML part content. Job alert emails contain:
- Job title
- Company name
- Location
- View job URL (LinkedIn: `/jobs/view/<JOB_ID>/`, EY: `/job/<slug>/<JOB_ID>/`)

Jobs are separated by dashes and `\r\n`.

## Step 3: Fetch Full JDs

### LinkedIn Guest API (no cookies needed)

```bash
curl -sL --max-time 15 -o /tmp/li_job.html "https://www.linkedin.com/jobs-guest/jobs/api/jobPosting/<JOB_ID>" && \
python3 -c "
import re, html
with open('/tmp/li_job.html','r',errors='ignore') as f:
    content = f.read()
if not content or len(content) < 500:
    print('EMPTY - job may be expired')
else:
    text = re.sub(r'<script[^>]*>.*?</script>', '', content, flags=re.DOTALL)
    text = re.sub(r'<style[^>]*>.*?</style>', '', content, flags=re.DOTALL)
    text = re.sub(r'<[^>]+>', ' ', text)
    text = html.unescape(text)
    lines = [l.strip() for l in text.split('\n') if l.strip()]
    print('\n'.join(lines[:150]))
"
```

- Returns ~66KB HTML per job
- Works for active postings only — expired jobs return empty
- No authentication/cookies needed
- No throttling at 1s intervals
- The actual JD text starts after LinkedIn's sign-in boilerplate (~line 15-20)

### EY / SuccessFactors Portal (browser required)

```bash
# curl returns only JS boilerplate (GTM scripts, cookie consent, translations)
# MUST use browser_navigate — the browser renders the JS
browser_navigate(url="https://careers.ey.com/ey/job/<slug>/<JOB_ID>/")
```

The browser_navigate accessibility snapshot contains the full JD:
- Job title (heading level=1)
- Location, salary, date
- Job description (heading level=2)
- Requisition ID
- Full responsibilities, requirements, qualifications
- Apply button

**Key:** SuccessFactors pages are JS-rendered SPA. curl gets GTM scripts
and cookie consent HTML only. The browser's accessibility tree extracts
the rendered content directly — no need for browser_snapshot after
navigate, the snapshot is in the navigate response.

### Terminal `&` in URL pitfall

When a URL contains `&` (e.g. `AI-&-Data` in EY job slug), the terminal
tool's security scan may flag it as backgrounding (`&`). Either:
- URL-encode the `&` as `%26` (works for EY/SuccessFactors)
- Use `browser_navigate` instead of `curl` for these URLs

## Step 4: Filter Relevant Jobs Before Fetching

From the EY email (10 jobs), skip obvious mismatches:
- Senior Business Developer BCM & Insurance — actuarial/insurance
- Wealth & Asset Management — finance/wealth
- Consulting Senior Actuarial (x2) — actuarial

Fetch only: Applied AI Engineer, AI & Data Consulting, Digital Solution Lead,
Senior Software Engineer.

From LinkedIn alerts (6 jobs), skip:
- Game Producer — gaming
- Staff Infrastructure Engineer - Models — hardware/semiconductor
- Staff Software Engineer (Cloud Provider Team) — IC devops

Fetch: R&D Director, AI Engineer, Data Platform Owner, Senior Product Owner.

## Step 5: Score with V5.0.0

Load `vaccancy-compatibility-scoring-engine` and `cv-repository` skills.
Read CV database from `~/CV_REPOSITORY_DATABASE.md`.
Apply hard filters, then 7-parameter scoring for each JD.

## Session Results (2026-08-02)

13 vacancies evaluated from 2 emails (EY + LinkedIn):
- 4 APPLY (score >= 75): Clarivate Director Delivery (81), NTT DATA AI Engineer (78), EY AI Engineer Manager (78), EY Digital Solution Lead AD (77)
- 2 APPLY SELECTIVELY (65-74): EY AI Engineer Senior (74), Clarivate Senior PO (72)
- 4 SKIP (score < 65): EY Backend SWE (51), HireRight Data Platform Owner (53)
- 5 DISQUALIFIED: EY Associate (below P-3), GRUNDFOS R&D Director (functional mismatch), MSD Digital Ops (below P-3), Prohuman Technical Director (expired), Performlabs Director (expired)