# LinkedIn Browser Automation — Channels 2 & 4

This reference covers using Chrome browser tools to search LinkedIn directly. Web search can only see what LinkedIn exposes to search engine crawlers — a small fraction of listings and almost none of the recruiter/HM posts. Direct browser browsing unlocks the full LinkedIn experience.

> **Tested March 2026.** Channel 2 (Jobs) uses standard HTML DOM — JavaScript extraction works. Channel 4 (Content Search) uses obfuscated CSS classes and SVG rendering — **JavaScript selectors DO NOT work**. Use `find` and `read_page` (accessibility tree) instead. These are stable across LinkedIn redesigns.

## When to Use Browser vs Web Search

| Situation | Method |
|-----------|--------|
| Chrome browser tools available (e.g., `navigate`, `computer`, `read_page`, `javascript_tool`) | **Use browser** — far more results, full JDs, recruiter posts |
| No browser tools available | Fall back to web search (current behavior), note limitation |
| LinkedIn shows CAPTCHA or login wall | Stop browser attempts, fall back to web search, log the error |

**How to detect browser availability**: Check if you have access to Chrome MCP tools like `navigate`, `computer`, `read_page`, `javascript_tool`, `find`. If these exist in your tool list, use browser mode for Channels 2 and 4.

## Pre-Flight Checks

Before starting LinkedIn browser searches:

1. **Get tab context** via `tabs_context_mcp` to see available tabs
2. **Create a new tab** via `tabs_create_mcp` (don't reuse the Google Sheet tab)
3. **Navigate to linkedin.com** and take a screenshot to check login status
4. **If not logged in**: Tell the user "LinkedIn requires login — please sign in to LinkedIn in this browser tab, then let me know when ready." Never attempt to enter credentials on the user's behalf.
5. **If logged in**: Proceed with searches

---

## Channel 2: LinkedIn Jobs — Browser Workflow

### Step 0: Freshness Sweep (NEW in v16.5 — Do This FIRST)

Before running the standard keyword search, do a quick freshness sweep to catch roles posted in the last 1-6 hours. Being applicant #1-10 instead of #300+ dramatically improves callback odds.

**How it works**: LinkedIn's `f_TPR` URL parameter controls the time filter in seconds:
- `f_TPR=r3600` = past 1 hour (3,600 seconds)
- `f_TPR=r7200` = past 2 hours
- `f_TPR=r21600` = past 6 hours
- `f_TPR=r86400` = past 24 hours (what "Past 24 hours" filter uses)
- `f_TPR=r604800` = past week (standard default)

**Freshness sweep URL template**:
```
https://www.linkedin.com/jobs/search/?keywords={query}&location={location}&f_TPR=r3600&f_E=4&sortBy=DD
```

**Procedure**:
1. For each of your top 3 keyword matrix queries, run the search with `f_TPR=r3600` (past hour)
2. If 0 results, widen to `f_TPR=r7200` (past 2 hours), then `f_TPR=r21600` (past 6 hours)
3. Any role found in the freshness sweep gets a **+10 IP "early bird" boost** (on top of the standard +5 recency bonus from scoring.md)
4. Flag these roles in the Action Pack as "POSTED <{X}h AGO — apply immediately for early-applicant advantage"
5. After the freshness sweep, proceed with the standard `f_TPR=r604800` search for broader coverage

**Time budget**: 2-3 minutes (3 queries × 1 page each). This is fast because fresh results are few.

**Why this works**: LinkedIn sorts applicants partly by recency. A resume submitted within the first hour gets significantly more recruiter attention than one submitted after 500+ applicants. Combined with our scoring system, this means a Fit 75 role found in its first hour may be more actionable than a Fit 85 role posted 5 days ago with 400 applicants.

**Dedup against standard search**: Roles found in the freshness sweep must be deduped against the standard search results (Step 1-3). If the same role appears in both, keep the freshness-sweep version (it retains the early-bird boost).

### Step 1: Build Search URLs

Construct LinkedIn Jobs search URLs using the keyword matrix from Phase 2B:

**Standard keyword search** (broad — finds roles across all companies):
```
https://www.linkedin.com/jobs/search/?keywords={query}&location={location}&f_TPR=r604800&f_E=4&sortBy=DD
```

**Company-filtered search** (targeted — finds roles at a specific company):
```
https://www.linkedin.com/jobs/search/?keywords={query}&f_C={company_ids}&geoId=103644278&f_E=4%2C5&sortBy=DD
```
Use this when the company is in `every_run` or `every_other` cadence AND has LinkedIn company IDs in `preferences.linkedin_company_filters`. For priority companies (especially those with referrals), run BOTH a company-filtered search AND the standard keyword search.

> **See `references/linkedin-company-filters.md`** for the full company ID registry, verification steps, and usage guidelines. That file is the authoritative source for all `f_C` mappings.

Key URL parameters:
- `keywords=` — URL-encoded search query
- `location=` — e.g., "San Francisco Bay Area", "United States"
- `f_C=` — Company ID filter. Look up in `references/linkedin-company-filters.md`
- `geoId=` — Geographic filter (103644278 = United States, 92000000 = Worldwide)
- `f_TPR=r604800` — past week. Use `r2592000` for past month. For freshness sweep see Step 0.
- `f_E=4` — mid-senior level. Use `f_E=5` for director. Use `f_E=4%2C5` for both.
- `sortBy=DD` — sort by most recent

### Step 2: Navigate and Extract Results

For each search query:

1. **Navigate** to the search URL
2. **Wait 3 seconds** for results to render
3. **Take a screenshot** to verify results loaded and confirm login state

4. **Extract job cards** using JavaScript (tested March 2026 — selectors work on Jobs pages):

```javascript
const cards = document.querySelectorAll('.job-card-container, .jobs-search-results__list-item');
const jobs = [];
cards.forEach(card => {
  const titleEl = card.querySelector('.job-card-list__title, .job-card-container__link span');
  const companyEl = card.querySelector('.job-card-container__primary-description, .artdeco-entity-lockup__subtitle');
  const locationEl = card.querySelector('.job-card-container__metadata-item, .artdeco-entity-lockup__caption');
  const linkEl = card.querySelector('a[href*="/jobs/view/"]');
  const timeEl = card.querySelector('time');
  const title = titleEl?.textContent?.trim();
  const company = companyEl?.textContent?.trim();
  const location = locationEl?.textContent?.trim();
  const link = linkEl?.href?.split('?')[0];
  const jobId = link?.match(/\/jobs\/view\/(\d+)/)?.[1];
  const postedDate = timeEl?.getAttribute('datetime') || timeEl?.textContent?.trim() || null;
  // Salary is embedded in card text — parse from full text
  const cardText = card.textContent;
  const salaryMatch = cardText.match(/\$[\d,]+[kK]?(?:\/yr|\/hr)?(?:\s*-\s*\$[\d,]+[kK]?(?:\/yr|\/hr)?)?/);
  const salary = salaryMatch ? salaryMatch[0] : null;
  if (title) jobs.push({ title, company, location, link, jobId, postedDate, salary });
});
JSON.stringify(jobs, null, 2);
```

**If JavaScript extraction returns 0 results** (LinkedIn may have changed the DOM), fall back to:
- Use `find` with query "job title and company name" to locate job cards
- Use `read_page` to get the accessibility tree — more stable than CSS selectors

5. **Paginate** — LinkedIn shows ~25 jobs per page but lazy-loads them:
   - **First**: Scroll down the page (`scroll` action, coordinate in the left panel area, 8-10 ticks down) to trigger lazy loading. Initial load shows ~7 cards; full scroll reveals ~23-25.
   - **Then**: Click page buttons. Use `button[aria-label="Page 2"]` or the `find` tool to locate "Page 2" / "Next" buttons. URL also supports `&start=25` for page 2, `&start=50` for page 3.
   - **Wait 3 seconds** after each pagination for new results to load.
   - FAST_TRACK: 2 pages. FULL_ANALYSIS: 3 pages.

### Step 3: Read Promising JDs

For roles where the title/level/company look relevant:

1. **Click the job card** via JavaScript (`cards[i].querySelector('a[href*="/jobs/view/"]').click()`) or navigate to `/jobs/view/{jobId}`
2. **Wait 2 seconds** for the JD panel to load on the right side
3. **Extract the full JD** (tested March 2026 — these selectors work):

```javascript
const jdEl = document.querySelector('.jobs-description__content, .jobs-box__html-content');
const title = document.querySelector('.job-details-jobs-unified-top-card__job-title, .t-24')?.textContent?.trim();
const company = document.querySelector('.job-details-jobs-unified-top-card__company-name')?.textContent?.trim();
const details = document.querySelector('.job-details-jobs-unified-top-card__primary-description-container')?.textContent?.trim();
const jdText = jdEl?.textContent?.trim();
// Check for Easy Apply (lower friction = higher Opportunity Score)
const hasEasyApply = !!document.querySelector('.jobs-apply-button');
const applyBtnText = document.querySelector('.jobs-apply-button, [class*="jobs-apply"]')?.textContent?.trim();
const isEasyApply = applyBtnText?.includes('Easy Apply') || false;
JSON.stringify({ title, company, details, jd: jdText?.substring(0, 3000), isEasyApply });
```

**Known issue**: `company` may return empty string via `.job-details-jobs-unified-top-card__company-name`. If empty, extract company from the `details` text (format: "Company Name · Location · Posted date") or from the job card data you already extracted.

4. **Extract salary** — look for pill/badge elements above the JD. Salary appears as text like "$165K/yr - $190K/yr" in the details container. Parse from `details` text.
5. **Parse posted date** from `details` text (format: "Location · X hours/days ago · N applicants")

### Step 4: Rate Limiting

LinkedIn throttles automated browsing, but testing shows moderate activity is tolerated:

- **Wait 3 seconds between page navigations** (use `wait` action). This was sufficient in testing — no throttling observed at this pace.
- **Maximum 5 search queries per session** — prioritize by Tier
- **Maximum 15 JD reads per session** — JD reads are lightweight (just loading the right panel)
- **STOP IMMEDIATELY if you see**:
  - CAPTCHA challenge or "verify you're human"
  - "We don't recognize this device"
  - "Unusual activity detected"
  - Blank pages or unexpected redirects to login
- If rate-limited: Log the error, report what you found so far, move on to other channels. Never retry aggressively.
- **Account safety note**: These limits protect the user's LinkedIn account. Err on the side of fewer actions. A session that extracts 50 jobs across 2-3 queries with 10-15 JD reads is a good target.

---

## Channel 4: LinkedIn Recruiter/HM Posts — Human-Like Browser Workflow

This is the highest-value channel. Recruiter and hiring manager posts represent the warmest leads because the poster is actively looking for candidates and has publicly signaled their intent. Response rates to post-based outreach are 3-5x higher than cold outreach.

> **CRITICAL**: LinkedIn's content search page uses **obfuscated CSS classes** (hashed like `_0eee3437`) and renders post content as **SVGs**, not standard HTML text. This means `document.querySelectorAll` with CSS class selectors **will always return 0 results**. You MUST use `find` and `read_page` (accessibility tree tools) for Channel 4 extraction. These tools read the accessibility layer which LinkedIn maintains regardless of their rendering approach.

**Design principle**: Ch4 should mirror how a real person browses LinkedIn for hiring signals — NOT jump mechanically to keyword search URLs. Follow this natural browsing flow in order.

---

### Step A: Home Feed Scan (DO THIS FIRST — warmest leads)

Navigate to `https://www.linkedin.com/feed/`.

Your connections' hiring posts surface here organically. These are the **warmest possible leads** because you already have a 1st-degree connection to the poster. A human always checks their feed first.

**Procedure:**
1. Navigate to LinkedIn feed
2. Wait 4 seconds for content to render
3. Take a screenshot to confirm feed loaded
4. Use `find` to scan for hiring-related posts:
   ```
   find(query="post about hiring or open role or join my team or growing the team", tabId=...)
   ```
5. Scroll down (4-6 scroll cycles, 3 sec wait between each):
   ```
   scroll(coordinate=[500, 400], scroll_direction="down", scroll_amount=8, tabId=...)
   ```
6. After each scroll, run `find` again to catch newly loaded posts
7. For any hiring-related post, use `read_page` with the post's ref_id to get full details

**What to look for**: Posts mentioning "hiring", "open role", "join my team", "growing the team", "we're looking for", "excited to announce", "new headcount", "backfill". Also watch for people resharing job postings.

**Time budget**: 2-3 minutes. This step often catches posts that keyword search misses because they use informal language.

---

### Step B: Notifications Check (Inbound Signals)

Navigate to `https://www.linkedin.com/notifications/`.

Scan for inbound interest signals — these are leads where someone is already reaching out to YOU:
- Recruiters viewing your profile (shows as "X from Company viewed your profile")
- Hiring managers engaging with your content (likes, comments)
- Connection requests from talent acquisition or recruiting teams
- Messages from recruiters (note: can't read full messages without clicking in)

**Procedure:**
1. Navigate to notifications page
2. Wait 3 seconds
3. Take a screenshot
4. Use `read_page` to scan notification items
5. Extract any relevant recruiter/HM names, companies, and profile URLs
6. These become immediate CRM contacts with `source: "inbound_signal"` and `path_strength: "strong"`

**Time budget**: 1 minute. Quick scan — don't deep-dive unless something is clearly relevant.

---

### Step C: Targeted Content Search (Keyword-Based)

This is the structured keyword search. Build content search URLs:

```
https://www.linkedin.com/search/results/content/?keywords={search_term}&datePosted=%22past-week%22&sortBy=%22date_posted%22
```

**Expanded search terms** (use variety — don't repeat the same pattern):

Role-specific terms:
- `"hiring data science manager"`
- `"looking for" "data science" "manager" OR "lead"`
- `"open role" "analytics" "senior manager"`
- `"hiring" "head of data science"`

Action-oriented terms:
- `"we're hiring" "data science" "Bay Area" OR "remote"`
- `"join my team" "analytics" OR "data science"`
- `"growing my team" "data science" OR "analytics"`

Urgency signals:
- `"backfill" "data science" OR "analytics"`
- `"immediate hire" "data" OR "analytics" "manager"`
- `"just opened" "data science" OR "analytics"`

Team-building signals:
- `"building a team" "data science" OR "analytics"`
- `"new team" "data" "hiring"`
- `"expanding" "data science" OR "analytics" "team"`

**Scope:**
- FAST_TRACK: Top 3 search terms, first 10 posts each.
- FULL_ANALYSIS: All terms (rotate through at least 8-10), first 20 posts each.

**For each search term:**
1. Navigate to the content search URL
2. Wait 4 seconds for feed content to render (content search is slower than job search)
3. Take a screenshot to visually confirm posts loaded
4. Extract post authors and titles using `find`:
   ```
   find(query="hiring post author name and job title", tabId=...)
   ```
5. Extract post content using `find`:
   ```
   find(query="post text content about hiring or looking for candidates", tabId=...)
   ```
6. For detailed extraction, use `read_page` with the post's ref_id:
   ```
   read_page(tabId=..., ref_id="ref_85", depth=8, max_chars=5000)
   ```

**Post structure in accessibility tree** (pattern observed March 2026):
```
listitem [ref_N]           ← Each post is a listitem inside main
  heading "Feed post"      ← Confirms this is a post
  link → author profile URL (/in/username/)
    generic → author name
    generic → author title/headline
  generic → post text content (the main body)
  link → reactions count
  button → Like/Comment/Repost/Send actions
```

**Scroll to load more** (infinite scroll — no pagination buttons):
1. After extracting visible posts, scroll down:
   ```
   scroll(coordinate=[500, 400], scroll_direction="down", scroll_amount=8, tabId=...)
   ```
2. Wait 3 seconds for new posts to render
3. Take a screenshot to see newly loaded posts
4. Run `find` again to extract new posts
5. Repeat until target post count reached or no new content
6. Typically 3-4 scroll cycles = 10-15 posts. For FA (20 posts), do 5-6 cycles.

**De-duplication**: Posts may repeat across scroll loads. Track seen author+text combinations.

---

### Step D: Target Company Page Browsing

For your top 3-5 target companies (pull from `learning_history.conversion_rates.by_company` — companies with `interviews >= 1` plus any active networking targets), browse their LinkedIn company pages.

**Procedure:**
1. Navigate to `https://www.linkedin.com/company/{company-slug}/posts/`
2. Wait 3 seconds
3. Take a screenshot
4. Use `find` to look for hiring-related posts from the company page
5. Scroll through 2-3 cycles to see recent posts
6. Companies often announce new roles through official LinkedIn posts BEFORE they hit the career page — this catches roles at the "just opened" stage

**FULL_ANALYSIS only** — skip in FAST_TRACK mode.

---

### Step E: My Network / Job Changes (Referral Mining)

Navigate to `https://www.linkedin.com/mynetwork/`.

**What to look for:**
- **Recent job changes**: Someone who just joined a target company = warm referral source. They're still in the "excited about new job" phase and more likely to refer.
- **New hiring managers**: Someone who just got promoted to a management role = potential warm lead with new headcount.
- **"People you may know"**: Occasionally surfaces relevant connections at target companies.

**Procedure:**
1. Navigate to My Network page
2. Wait 3 seconds
3. Take a screenshot
4. Use `read_page` to scan for job change notifications and suggested connections
5. Extract relevant contacts — add to CRM with `source: "network_change"` and appropriate `path_strength`

**FULL_ANALYSIS only** — skip in FAST_TRACK mode.

---

### Step F: Convert Posts to Leads

For each relevant hiring post found across Steps A-E:

1. **Classify the poster**:
   - **Hiring Manager** (has VP/Director/Sr Manager/Head of title at the company) → +15 IP boost
   - **Internal Recruiter** (title contains "Recruiter", "Talent Acquisition") → +10 IP boost
   - **Team member** ("we're growing!", "join my team") → +5 IP boost
   - **Job board/aggregator** (e.g., "City of Westminster jobs", "JobNirvana") → +0 IP boost (lower value)

2. **Extract role details**: title, level, team, location, salary mentions, urgency signals ("immediately", "backfill", "ASAP")

3. **Find the formal posting**: Check if the post includes a "View job" link (visible in the accessibility tree as a link element). If not, search the company's careers page.

4. **Generate post-referencing outreach draft** — this is the key deliverable from Channel 4:

```
Hi {poster_name},

I saw your post about the {role} on the {team} team at {company}.
With {X}+ years leading {domain} teams — including {specific_match} —
this looks like a strong fit. I'd love to learn more about what you're
looking for. Would you be open to a quick chat?

Best,
{candidate_name}
```

5. **Add poster as a CRM contact** with:
   - `path_strength`: "moderate" (post-based outreach converts well) — upgrade to "strong" for Step A (feed) and Step B (notifications) leads
   - `outreach_status`: "pending"
   - `source`: "linkedin_post" (or "linkedin_feed", "inbound_signal", "network_change" based on step)
   - `profile_url`: extracted from the accessibility tree (`/in/{username}`)
   - `linked_roles`: [the role from the post]

### Step G: Rate Limiting

Be careful across ALL steps — LinkedIn monitors overall session activity, not just search queries:
- **Wait 3-4 seconds between page navigations**
- **Maximum 5-8 content searches per session** (Step C)
- **Scroll slowly** — 3 seconds between scroll actions
- **Total session time**: Keep Ch4 under 15 minutes to avoid triggering rate limits
- Stop on any CAPTCHA or unusual activity warnings
- A good session target: Steps A-E should yield 30-75 posts scanned total

---

## Handling Common Issues

### JavaScript Extraction Returns Empty (Channel 2)
LinkedIn Jobs page selectors are relatively stable, but if they break:
1. Take a screenshot to see the actual page
2. Use `find` with query "job title and company name" — this reads the accessibility tree
3. Use `read_page` to get the full tree structure
4. Adapt extraction based on what you find

### JavaScript Extraction Returns Empty (Channel 4)
**This is EXPECTED.** LinkedIn's content search uses obfuscated rendering. Always use `find` and `read_page` for Channel 4:
1. Use `find(query="hiring post author name and job title")` for authors
2. Use `find(query="post text content about hiring")` for post content
3. Use `read_page(ref_id=..., depth=8)` for detailed single-post extraction
4. **Never rely on CSS selectors** for the content search page

### Session Timeout
If pages start redirecting to login mid-run, ask the user to re-authenticate. Resume where you left off.

### No Browser Tools Available
If Chrome browser tools aren't in the tool list, fall back gracefully:
- Channel 2: Use web search `site:linkedin.com/jobs "{query}"` — limited but better than nothing
- Channel 4: Use web search `site:linkedin.com "hiring" "{query}"` — very limited, most posts won't be indexed
- Note the limitation clearly in the Channel Completion Log

---

## Integration with Pipeline

### Channel 2 → Phase 4 (JD reading):
Each extracted job feeds into the standard JD evaluation pipeline with: title, company, location, URL, jobId, salary, posted date.

### Channel 4 → Phase 5.5 (networking):
Post-based leads automatically include a networking contact (the poster). These get IP boosts and post-referencing outreach drafts. Add contacts to CRM.

### Channel Completion Log format:
```
Ch2 (LinkedIn Jobs):        [✓] — {X} roles found via browser | {Y} queries | {Z} pages | {notes}
Ch4 (Recruiter/HM Posts):   [✓] — {X} leads found via browser (find/read_page) | {Y} searches | {Z} scroll cycles | {notes}
```

If browser unavailable:
```
Ch2 (LinkedIn Jobs):        [⚠] — {X} roles via web search (browser unavailable — limited results)
Ch4 (Recruiter/HM Posts):   [⚠] — {X} leads via web search (browser unavailable — most posts not indexed)
```
