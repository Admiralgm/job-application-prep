# Company IT-Stack Discovery (CTO/CIO applications, technical due diligence)

Proven on Zepter International Group CTO case-study research (Aug 2026). When the
application/case study requires understanding a target company's IT environment,
these sources beat generic "company IT" searches. Order roughly by value:

## 1. Job postings = best IT-stack source
LinkedIn guest job pages are server-rendered and fetchable via curl with a browser
UA — no login needed:
```bash
curl -sL "https://rs.linkedin.com/jobs/view/<slug-id>" \
  -H "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0 Safari/537.36"
```
then strip tags with the standard python3 HTML-to-text pipeline. Search
`site:linkedin.com/jobs <company> <role>` for CTO, IT Director, DevOps, developer,
ERP consultant. Job ads name the ERP, cloud provider, and roadmap. Phrases like
"goal to implement Kubernetes/Terraform" mean ASPIRATIONAL, not current — that
distinction matters for case studies.

## 2. Glassdoor job listings reveal legacy stack
`site:glassdoor.com <company> jobs` — e.g. "MS DYNAMICS NAV CONSULTANT" +
"ASP.NET APPLICATION DEVELOPER" in Poland revealed Dynamics NAV heritage (the
predecessor of the current Dynamics 365 Business Central) and an ASP.NET web
platform. Legacy job titles = migration history.

## 3. Cookie analysis fingerprints the web stack
Read the site's cookie-consent details (Cookiebot) or Set-Cookie headers:
- `ASP.NET_SessionId` / `__RequestVerificationToken` / `owin.authentication` → ASP.NET MVC/OWIN
- `PHPSESSID` → PHP
- `_ga` → Google Analytics; `smc_uid` → cart-recovery vendor
The Cookiebot consent dialog on the page itself lists every cookie + purpose —
grab it from the rendered page or the raw HTML.

## 4. ASN lookup reveals self-hosted infrastructure
```bash
curl -s "https://stat.ripe.net/data/as-overview/data.json?resource=AS<num>"
```
→ holder name (e.g. "ZEPTERIT-AS Zepter IT Sp. z o.o."). Find the ASN first via
`ipinfo.io/AS<num>` (needs token for JSON; the HTML page works) or bgpview.io.
An own ASN + own data center = in-house hosting, not SaaS-only.

## 5. Footer copyright = entity-discovery signal
"© Copyright by Zepter IT" on every country site → the in-house IT company.
Then pull registry data (KRS/NIP/REGON for Poland, EMIS, DnB) for founding year
and headcount. Zepter IT Sp. z o.o. (Warsaw, est. 2003, ~17 staff) coordinates
Group IT — the "established IT organisation in Poland" the HR email referenced.

## 6. Revenue/employee estimates
ZoomInfo/EMIS/RocketReach snippets via Serper give revenue/employee estimates —
label as estimates, cross-check against the official company profile page.

## 7. Pitfall: Jina Reader on JS-rendered corporate sites
Jina Reader on a JS SPA (e.g. shop.zepter.com) returns only nav + cookie banner;
compact mode truncates to ~5KB of nav. For the real body, use browser_navigate
(Camoufox) — the accessibility snapshot renders full content including discount
tables, registration flows, and partner-benefit sections. The company profile
page's own numbers (factories, m², consultants) came from the browser snapshot,
not from curl/Jina.

## 8. Case-study-specific extras
- The HR email itself is a source: it names the ERP (Microsoft Business Central),
  the IT org location (Poland), and the platform list (ZepterClub, e-commerce).
- Register on the target's customer/sales platform if HR asks — the registration
  flow (fields, recommender/network-tree field, tier tables) is itself evidence
  of the business model and platform capabilities.
- Current CTO/leadership: check LinkedIn company page posts for recent C-level
  appointments (e.g. "welcome X as our new CTO") — names the incumbent and the
  org's hiring pattern.
