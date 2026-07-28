# LinkedIn Search Results JD Extraction — curl + Chrome Cookies Technique

> **Date:** 2026-07-24 (updated with session 2 findings)
> **Context:** Extracting full JDs from LinkedIn job search results pages (not the Jobs Tracker).
> **Problem:** LinkedIn throttles dynamic JD content rendering in Chrome after ~3 rapid navigations. AppleScript `execute javascript` returns only page chrome (nav bar, footer) with no JD text.

## Technique: Multi-Method Fallback Chain

### Method 1 (PRIMARY): curl + Chrome Cookies

Reliable for ALL jobs. No throttling. Works at scale (17+ JDs in one batch).

#### Step 1: Extract cookies from Chrome via AppleScript

```applescript
tell application "Google Chrome"
  -- find linkedin.com tab
  set wIdx to 0
  set tIdx to 0
  repeat with i from 1 to (count of windows)
    repeat with j from 1 to (count of tabs of window i)
      if (URL of tab j of window i) contains "linkedin.com" then
        set wIdx to i
        set tIdx to j
        exit repeat
      end if
    end repeat
    if wIdx > 0 then exit repeat
  end repeat
  set jsCode to "document.cookie"
  set cookieResult to execute tab tIdx of window wIdx javascript jsCode
  return cookieResult
end tell
```

Write the cookie string to a file. Cookies contain semicolons — do NOT inline them in curl `-H "Cookie: ..."` (special chars break the shell). Use Netscape cookie file format:

```python
cookie_lines = cookie_str.split('; ')
netscape_cookie = "# Netscape HTTP Cookie File\n"
for c in cookie_lines:
    parts = c.split('=', 1)
    if len(parts) == 2:
        name, value = parts
        netscape_cookie += f".linkedin.com\tTRUE\t/\tFALSE\t0\t{name}\t{value}\n"
# write to /tmp/cookies.txt
```

#### Step 2: Fetch JDs via LinkedIn guest API

```bash
curl -s --max-time 10 -b /tmp/cookies.txt \
  "https://www.linkedin.com/jobs-guest/jobs/api/jobPosting/{JOB_ID}" \
  2>/dev/null
```

Returns ~66KB HTML per job. Strip tags with Python regex:

```python
text = re.sub(r'<[^>]+>', '\n', html)
text = re.sub(r'\n\s*\n', '\n', text)
text = re.sub(r'[ \t]+', ' ', text).strip()
```

The "About" section starts after LinkedIn's sign-in/login boilerplate. Search for `About the job`, `About`, or the company name to find the JD start.

#### Step 3: Batch extraction

Loop through job IDs with `time.sleep(1)` between requests. No rate limiting observed at 1s intervals. All 13 remaining JDs fetched in ~20 seconds total.

### Method 2 (FALLBACK): AppleScript + Chrome navigation

Works for the first ~3 jobs, then LinkedIn throttles. Useful when you need the search page job list (IDs + titles + companies + locations).

```applescript
-- Navigate to job view page
-- Wait 5-7 seconds
-- Extract: document.querySelector('main').innerText.substring(0, 8000)
```

### Method 3: Search page job list extraction

Extract all visible job IDs from a search results page:

```javascript
var cards = document.querySelectorAll('a[href*="/jobs/view/"]');
var seen = {};
var results = [];
for (var i = 0; i < cards.length; i++) {
  var match = cards[i].href.match(/\/jobs\/view\/(\d+)/);
  if (match && !seen[match[1]]) {
    seen[match[1]] = true;
    // get title, company, location from card
    results.push(match[1] + ' | ' + title + ' | ' + company + ' | ' + location);
  }
}
```

### Pagination

LinkedIn search results paginate with `&start=25` appended to the search URL. Page 1 = `start=0` (default), page 2 = `start=25`, etc. Each page shows ~8-10 jobs.

**⚠️ Pagination reliability (updated 2026-07-24 session 2):** Navigating Chrome to `&start=25` does NOT reliably load page 2 job cards — LinkedIn's lazy-loading doesn't trigger on URL change alone. The page renders only the current job's detail. To get page 2 job IDs:
1. Navigate to the `&start=25` URL in Chrome
2. Scroll the page 2-3 times (`window.scrollBy(0, 3000)`) with 2s pauses
3. Extract job IDs via `querySelectorAll`

Even with scrolling, only ~5 new job cards may load (vs 8-10 on page 1). For comprehensive extraction, also try the guest API search endpoint, though it has proven unreliable for pagination (returns redirect HTML without job IDs). The most reliable approach is to extract all visible IDs from page 1, then use curl to fetch each JD individually.

## What Doesn't Work

- **Scrolling the search page** (`window.scrollBy(0, 2000)`) does NOT load more job cards — LinkedIn uses pagination, not infinite scroll on the search results page.
- **Clicking job cards in split view** — the right pane JD content was throttled/empty even after 8s wait. The split view requires authentication state that degrades under rapid navigation.
- **curl without cookies** — the guest API returns empty responses for most job IDs. Cookies from an authenticated Chrome session are required.
- **Inline cookie in curl `-H`** — special characters in the cookie string (especially `=`, `;`) break shell parsing. Always use `-b /tmp/cookies.txt` with Netscape format.
- **`/jobs/search-results/` URL format** — renders differently from `/jobs/search/` and doesn't expose job cards to `querySelectorAll`. The search-results format shows only 1 job card in Chrome (the currentJobId), while `/jobs/search/` shows 6+ cards. Always use `/jobs/search/` format for Chrome-based extraction. The search-results format is the one LinkedIn generates from in-app notifications — it's less useful for extraction.
- **Voyager API** (`/voyager/api/jobSearch/jobs`) — returns 0 results for job search queries. Confirmed non-functional for this purpose across multiple sessions.
- **`seeMoreJobPostings/search` pagination** — `start=25` and `start=50` return empty HTML (26 bytes). Only `start=0` returns job IDs. Do not rely on pagination via this endpoint.

## What Works (for finding MORE job IDs beyond what Chrome renders)

### Guest API `seeMoreJobPostings/search` (start=0 only)

The endpoint `https://www.linkedin.com/jobs-guest/jobs/api/seeMoreJobPostings/search?keywords=...&start=0` returns ~30KB HTML with `data-job-id` attributes. This reliably finds 10+ job IDs per query. Use it as a supplement to Chrome extraction when you need more IDs than the search page renders.

```python
import re, subprocess
url = "https://www.linkedin.com/jobs-guest/jobs/api/seeMoreJobPostings/search?keywords=KEYWORDS&geoId=GEOID&f_TPR=FILTER&start=0"
result = subprocess.run(["curl", "-s", "--max-time", "15", "-b", "/tmp/cookies.txt", url], capture_output=True, text=True)
ids = re.findall(r'data-job-id="(\d+)"', result.stdout)
```

**Limitations:** Only `start=0` works. `start=25` and higher return empty. The endpoint does not support pagination — you get one page of results only.

## Session Stats

### Session 1 (2026-07-24, Serbia jobs alert)
- Total jobs found: 17 (8 page 1 + 9 page 2)
- Full JDs extracted via Chrome AppleScript: 3 (first 3 only, then throttled)
- Full JDs extracted via curl + cookies: 14 (all remaining, zero failures)
- Total extraction time: ~3 minutes including navigation

### Session 2 (2026-07-24, IT PM semantic search)
- Total jobs found: 9 (page 1 only, pagination failed)
- Full JDs extracted via Chrome AppleScript: 2 (already scored from prior session)
- Full JDs extracted via curl + cookies: 7 (all remaining, zero failures)
- Pagination via `&start=25` failed in both Chrome and curl API
- Guest API search endpoint returned redirect HTML without job IDs

### Session 3 (2026-07-25, IT PM search — corrected URL)
- **URL format used:** `/jobs/search-results/` (user-specified) — rendered only 1 job card in Chrome
- **URL format reverted to:** `/jobs/search/` — rendered 6 job cards
- **Guest API `seeMoreJobPostings/search?start=0`:** Found 10 job IDs via `data-job-id` attributes (NEW finding — this endpoint DOES work for ID discovery, contradicting session 2's finding)
- **Guest API pagination:** `start=25` and `start=50` returned empty (26 bytes each) — confirmed no pagination support
- **Voyager API:** Returned 0 results — confirmed non-functional
- **Total unique IDs found:** 19 (6 from Chrome + 10 from guest API + 3 from URL landing postings)
- **Valid JDs fetched via curl:** 17 (2 returned empty/blocked content)
- **Key lesson:** The `/jobs/search-results/` format is the one LinkedIn generates from in-app notifications. It's less useful for extraction than `/jobs/search/`. When the user provides a search-results URL, navigate Chrome to it first (to establish the session), then switch to `/jobs/search/` format for extraction, or use the guest API for ID discovery.
