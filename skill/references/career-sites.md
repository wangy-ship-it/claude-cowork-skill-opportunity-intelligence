# Career Site Navigation Guide

This file contains tips for navigating specific company career sites. It grows over time as users search different companies. If a company isn't listed, use the General Approach section.

## General Approach

1. **Start with a web search**: `"{company name} careers site"` to find the correct URL
2. **Take a screenshot first**: Understand the layout before scripting
3. **Look for an API**: Many modern career sites use REST APIs behind their search UI. Check Network requests for JSON endpoints.
4. **Try JavaScript extraction**: Most sites render job cards as `<a>` elements with class names containing "card", "job", "result", or "listing". Try:
   ```javascript
   document.querySelectorAll('a[class*="card-"], a[class*="job-"], a[class*="result"]')
   ```
5. **Check for pagination**: Look for "next page" buttons, page numbers, or `start=` / `offset=` URL parameters
6. **Prefer canonical URLs**: Job URLs with a unique ID (e.g., `?pid=12345` or `/job/12345`) are more stable than search result URLs

## Microsoft

**Careers URL**: `https://careers.microsoft.com` (redirects to `apply.careers.microsoft.com`)

**Search URL pattern**:
```
https://apply.careers.microsoft.com/careers?query={search_term}&start={offset}&location={location}&sort_by=relevance&filter_distance=160&filter_include_remote=1
```

**Job URL pattern**: `https://apply.careers.microsoft.com/careers/job/{PID}`

**Key notes**:
- New careers site launched late 2025 — uses `apply.careers.microsoft.com`
- Shows 20 jobs per page; paginate with `start=0`, `start=20`, `start=40`
- Job cards are `a` tags with class containing `card-`. Extract PID from href: `/job/{PID}`
- Experience level filter dropdown exists but can be finicky — sometimes doesn't apply via URL params
- Locations include "+X more" — click into the job to see all locations
- The `get_page_text` on a job page gives you the full description including requirements, qualifications, and compensation
- JavaScript extraction pattern:
  ```javascript
  const cards = document.querySelectorAll('a[class*="card-"]');
  cards.forEach(card => {
    const pid = card.href.match(/job\/(\d+)/)?.[1];
    const text = card.textContent.trim();
    // Title is before "United States", posted date after "Posted"
  });
  ```

## Apple

**Careers URL**: `https://jobs.apple.com`

**Key notes**:
- Search with keywords and location filters
- Uses dynamic rendering — may need to wait for results to load
- Job IDs are numeric, URLs like `https://jobs.apple.com/en-us/details/{ID}/{slug}`
- Pagination via scrolling or page buttons

## Google / Alphabet

**Careers URL**: `https://www.google.com/about/careers/applications/`

**Key notes**:
- Clean search interface with keyword and location filters
- Uses URL parameters for search: `?q={query}&location={location}`
- Well-structured job cards
- Note: Some roles are at Alphabet subsidiaries (Waymo, DeepMind, etc.)

## Amazon

**Careers URL**: `https://www.amazon.jobs`

**Key notes**:
- Separate from AWS-specific roles
- Search by keyword and location
- Job IDs in URL paths
- Many roles list multiple locations

## Netflix

**Careers URL**: `https://jobs.netflix.com`

**Key notes**:
- Relatively small number of open roles (hundreds, not thousands)
- Clean search interface
- Teams/organizations clearly labeled
- "Open to remote" flag on some roles

## Meta / Facebook

**Careers URL**: `https://www.metacareers.com`

**Key notes**:
- Search by role type, location, and team
- URLs contain job IDs
- Many roles list as "Remote, US" with office options

## DoorDash

**Careers URL**: `https://careersatdoordash.com`

**Search URL pattern**:
```
https://careersatdoordash.com/job-search/?keyword={search_term}
```

**Job URL pattern**: `https://careersatdoordash.com/job-search/?id={job_id}` (7-digit IDs like 3313445)

**Key notes**:
- Proprietary ATS — no Greenhouse/Ashby/Lever API (all 4 ATS probes return 404)
- Search supports keyword and location filters; "Expand Filter Options" reveals department/function filters
- Job IDs are visible on the search results page under each listing
- Department "Analytics & Data Science" and Function "Data Science" are the key filters for DS roles
- **Dual URL/ID system**: Search results show `Job ID: 3313445` but individual job page URLs use a DIFFERENT ID: `/jobs/{slug}/7454878/`. These are NOT the same number. When cross-referencing, use the Job ID from search results.
- **Career site uses STRICT keyword matching** — unlike LinkedIn's fuzzy search. "data science" returns 1 result (Brazil only). "analytics" returns 13. "strategy" returns 61. Always try multiple keywords.
- **March 2026 status**: DS Analytics team in core verticals (GTM, Consumer Growth, Grocery, Drive) froze hiring. Ads & Promotions Analytics vertical still active. Multiple active roles: Sr Manager Ads & Promotions Analytics (3313445), Manager Rx Advertiser Analytics (3351334), Procurement Manager Analytics (3359010)
- LinkedIn company ID: `f_C=3205573` (verified March 2026). NOTE: Previously listed as `f_C=2520` — this was incorrect.
- **Title vocabulary** (v16.7): HIGH YIELD: "analytics", "insights", "strategy". LOW YIELD: "data science", "manager data". DoorDash titles most data roles under "Analytics" not "Data Science" — this is the single most important thing to know about searching their career site.
- **Team-level hiring status**: Ads & Promotions Analytics = active hiring | DS Analytics (Core) = freeze | Strategy & Ops = active hiring (61 results for "strategy")

## Salesforce

**Careers URL**: `https://careers.salesforce.com`

**Key notes**:
- Clean Workday-based ATS
- Good keyword search
- Location filters work well

## General ATS patterns

Many companies use the same Applicant Tracking Systems:

- **Workday**: URL contains `myworkdayjobs.com`. Clean search, good filters.
- **Greenhouse**: URL contains `greenhouse.io` or `boards.greenhouse.io`. API available at `/api/v1/`.
- **Lever**: URL contains `lever.co` or `jobs.lever.co`. Simple list layout.
- **iCIMS**: URL contains `icims.com`. Older UI, may need more manual navigation.
- **Taleo**: URL contains `taleo.net`. Legacy system, clunky but functional.

## LinkedIn (the company) — Career Site

**Careers URL**: `https://careers.linkedin.com`

**Redirect pattern**: LinkedIn's career site does NOT have inline search. The "Search Jobs" button redirects to LinkedIn's own job board with company filters pre-applied. Use company-filtered LinkedIn search instead.

**Key notes**:
- Proprietary ATS — no Greenhouse/Ashby/Lever API
- Must search via LinkedIn job board with `f_C` company filter
- See `references/linkedin-company-filters.md` for LinkedIn's company IDs and the redirect pattern details
- Company page: `https://www.linkedin.com/company/linkedin/jobs/`

## LinkedIn Job Board (general — all companies)

**URL**: `https://www.linkedin.com/jobs/`

**Search URL pattern**:
```
https://www.linkedin.com/jobs/search/?keywords={query}&location={location}&f_TPR=r604800&position=1&pageNum={page}
```

**Company-filtered search**: Use `f_C={company_ids}` to filter to a specific company. See `references/linkedin-company-filters.md` for the full company ID registry, URL template, verification steps, and known wrong IDs.

**Key notes**:
- LinkedIn requires login for full results — ensure you're logged in before searching
- Job cards are rendered in a scrollable list on the left panel
- Each card shows: title, company, location, posted date (relative), "Easy Apply" flag
- Clicking a card loads the full JD in the right panel
- Pagination via scrolling or explicit page buttons at bottom

**JavaScript extraction pattern**:
```javascript
const cards = document.querySelectorAll('.job-card-container');
const jobs = [];
cards.forEach(card => {
  const title = card.querySelector('.job-card-list__title')?.textContent?.trim();
  const company = card.querySelector('.job-card-container__primary-description')?.textContent?.trim();
  const location = card.querySelector('.job-card-container__metadata-item')?.textContent?.trim();
  const link = card.querySelector('a.job-card-container__link')?.href;
  const jobId = link?.match(/\/jobs\/view\/(\d+)/)?.[1];
  if (title && company) jobs.push({ title, company, location, link, jobId });
});
JSON.stringify(jobs, null, 2);
```

**Rate limiting**:
- LinkedIn throttles automated browsing — space searches 5-10 seconds apart
- If you see a CAPTCHA or "unusual activity" warning, stop immediately
- Limit to 3-5 searches per session to avoid triggering rate limits
- Using the logged-in browser session (via Chrome extension) is more reliable than API calls

**New company detection**:
- Compare each result's company name against `target_companies` in preferences
- Companies NOT in the target list are potential Tier 3 discoveries
- Only flag as discovery if the role title/level matches the candidate profile

## Boolean/ATS Board Search

**Approach**: Use Google search with `site:` operators to find roles across multiple companies' ATS pages simultaneously.

**Google Boolean query templates**:
```
site:boards.greenhouse.io OR site:jobs.lever.co "data science" "manager" "San Francisco" OR "remote"
site:boards.greenhouse.io OR site:jobs.lever.co OR site:jobs.ashbyhq.com "analytics" "senior manager" OR "director"
site:boards.greenhouse.io "machine learning" "lead" OR "head of" "Bay Area" OR "remote"
site:jobs.lever.co "data science" "manager" OR "lead" 2026
site:jobs.ashbyhq.com "analytics" OR "data science" "manager"
site:smartrecruiters.com "data science" "manager" "San Francisco"
```

**ATS URL patterns for extraction**:

### Greenhouse
- **Board URL**: `https://boards.greenhouse.io/{company}`
- **Job URL**: `https://boards.greenhouse.io/{company}/jobs/{job_id}`
- **API endpoint** (public): `https://boards-api.greenhouse.io/v1/boards/{company}/jobs`
  - Returns JSON with all open positions
  - Each job has: `id`, `title`, `location.name`, `updated_at`, `absolute_url`
  - Example: `https://boards-api.greenhouse.io/v1/boards/anthropic/jobs`
- **Job detail API**: `https://boards-api.greenhouse.io/v1/boards/{company}/jobs/{job_id}`
  - Returns full JD content as HTML in the `content` field
- **Advantages**: Clean API, no rate limiting on public boards, full JD text available programmatically

### Lever
- **Board URL**: `https://jobs.lever.co/{company}`
- **Job URL**: `https://jobs.lever.co/{company}/{job_id}`
- **No public API** — must scrape the board page
- **JavaScript extraction**:
```javascript
const postings = document.querySelectorAll('.posting');
const jobs = [];
postings.forEach(p => {
  const title = p.querySelector('.posting-title h5')?.textContent?.trim();
  const location = p.querySelector('.posting-categories .sort-by-location')?.textContent?.trim();
  const team = p.querySelector('.posting-categories .sort-by-team')?.textContent?.trim();
  const link = p.querySelector('a.posting-title')?.href;
  if (title) jobs.push({ title, location, team, link });
});
JSON.stringify(jobs, null, 2);
```

### Ashby
- **Board URL**: `https://jobs.ashbyhq.com/{company}`
- **Job URL**: `https://jobs.ashbyhq.com/{company}/{job_id}`
- **API endpoint**: Check for `https://jobs.ashbyhq.com/api/non-user-graphql` (GraphQL)
- **Note**: Ashby sites are React-rendered — may need to wait for content to load before extraction
- Modern startups increasingly use Ashby (Notion, Linear, etc.)

### SmartRecruiters
- **Board URL**: `https://careers.smartrecruiters.com/{company}`
- **Job URL**: `https://careers.smartrecruiters.com/{company}/{job_id}`
- **API endpoint**: `https://api.smartrecruiters.com/v1/companies/{company}/postings`
  - Returns JSON with open positions

**Google search pagination**:
- First page: standard Google search URL
- Subsequent pages: add `&start=10`, `&start=20`, etc.
- Limit to first 2 pages (20 results) — beyond that, relevance drops significantly

**Tip**: When a Google result leads to a promising company not in your target list, check their full careers page (not just the one role) for additional relevant positions.

## LinkedIn Recruiter/HM Post Mining

**Approach**: Search LinkedIn's content/feed for posts by recruiters and hiring managers about open positions. These are the warmest possible leads.

**Search URL pattern**:
```
https://www.linkedin.com/search/results/content/?keywords={search_term}&datePosted=%22past-week%22&sortBy=%22date_posted%22
```

**Recommended search terms** (customize based on candidate profile):
- `"hiring data science manager"`
- `"looking for data science lead"`
- `"open role analytics manager"`
- `"growing my team" "data science"`
- `"hiring" "ML" "manager" "Bay Area"`
- `"we're hiring" "analytics" "senior"`

**Signal extraction from posts**:
When reading each post, extract:
1. **Poster role**: Is this person the hiring manager, a recruiter, or a team member? (Hiring manager posts are highest-value)
2. **Company name**: Where does the poster work?
3. **Role details**: What role/level/team are they hiring for?
4. **Urgency signals**: "immediately", "ASAP", "growing fast", "backfill"
5. **Direct contact**: The poster's LinkedIn profile URL (for outreach)
6. **Application method**: Does the post include a direct link to apply? Or say "DM me"?

**Converting posts to job leads**:
1. If the company is in `target_companies`: Search their careers page for the matching role
2. If the company is NOT in target list: Evaluate as a Tier 3 discovery
3. Generate a **post-referencing outreach draft**:
```
Hi {poster_name},

I saw your post about hiring a {role} on the {team} team at {company}.
I'm a {candidate_level} with {X}+ years in {domain} — {positioning_statement}.

I'd love to learn more about the role. Would you be open to a quick chat?

Best,
{candidate_name}
```

**Why post-based outreach works**:
- You're responding to a signal the person CHOSE to broadcast — they WANT to hear from candidates
- Referencing the specific post shows you're attentive and targeted (not spray-and-pray)
- Posts from hiring managers often bypass recruiter gatekeeping
- Response rates to post-based outreach are 3-5x higher than cold InMails

**Rate limiting**:
- LinkedIn content search is less aggressively throttled than job search, but still be careful
- Space searches 5-10 seconds apart
- Limit to 5-8 searches per session
- If prompted for CAPTCHA, stop immediately
