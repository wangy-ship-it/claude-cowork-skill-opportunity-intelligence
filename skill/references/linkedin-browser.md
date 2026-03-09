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

### Step 1: Build Search URLs

Construct LinkedIn Jobs search URLs using the keyword matrix from Phase 2B:

```
https://www.linkedin.com/jobs/search/?keywords={query}&location={location}&f_TPR=r604800&f_E=4&sortBy=DD
```

Key URL parameters:
- `keywords=` — URL-encoded search query
- `location=` — e.g., "San Francisco Bay Area", "United States"
- `f_TPR=r604800` — past week. Use `r2592000` for past month
- `f_E=4` — mid-senior level. Use `f_E=5` for director
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

## Channel 4: LinkedIn Recruiter/HM Posts — Browser Workflow

This is the highest-value channel. Recruiter and hiring manager posts represent the warmest leads because the poster is actively looking for candidates and has publicly signaled their intent. Response rates to post-based outreach are 3-5x higher than cold outreach.

> **CRITICAL**: LinkedIn's content search page uses **obfuscated CSS classes** (hashed like `_0eee3437`) and renders post content as **SVGs**, not standard HTML text. This means `document.querySelectorAll` with CSS class selectors **will always return 0 results**. You MUST use `find` and `read_page` (accessibility tree tools) for Channel 4 extraction. These tools read the accessibility layer which LinkedIn maintains regardless of their rendering approach.

### Step 1: Build Content Search URLs

```
https://www.linkedin.com/search/results/content/?keywords={search_term}&datePosted=%22past-week%22&sortBy=%22date_posted%22
```

Recommended search terms (customize from candidate profile keywords):
- `"hiring data science manager"`
- `"looking for" "data science" "manager" OR "lead"`
- `"open role" "analytics" "senior manager"`
- `"growing my team" "data science" OR "analytics"`
- `"we're hiring" "data science" "Bay Area" OR "remote"`
- `"join my team" "analytics" OR "data science"`
- `"hiring" "head of data science"`

FAST_TRACK: Top 3 search terms, first 10 posts each.
FULL_ANALYSIS: All terms, first 20 posts each.

### Step 2: Navigate and Extract Posts (Accessibility Tree Method)

For each search term:

1. **Navigate** to the content search URL
2. **Wait 4 seconds** for feed content to render (content search is slower than job search)
3. **Take a screenshot** to visually confirm posts loaded

4. **Extract post authors and titles** using `find`:
```
find(query="hiring post author name and job title", tabId=...)
```
This returns refs for author names (e.g., "Steen Whidden") and their titles (e.g., "Senior Technical Recruiter @ Google | AI & Research Data Science"). The `find` tool reliably extracts these even though CSS selectors fail.

5. **Extract post content** using `find`:
```
find(query="post text content about hiring or looking for candidates", tabId=...)
```
This returns the text content of hiring-related posts with their ref IDs.

6. **For detailed extraction of a specific post**, use `read_page` with the post's ref_id:
```
read_page(tabId=..., ref_id="ref_85", depth=8, max_chars=5000)
```
This reveals the full accessibility tree for that post, including:
- Author name and profile URL (in `link` elements with `href="/in/{username}"`)
- Author title/headline (in `generic` elements after the name)
- Post text content (in a `generic` element — usually the longest text block)
- Engagement metrics (reactions, comments, reposts)
- Hashtags (in `link` elements with `href` containing `HASH_TAG_FROM_FEED`)
- "View job" links if the post includes a formal job card

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

### Step 3: Scroll to Load More Posts (Infinite Scroll)

Channel 4 uses **infinite scroll** — there are NO pagination buttons like Channel 2.

**Scroll procedure**:
1. After extracting visible posts, use `scroll` action:
   ```
   scroll(coordinate=[500, 400], scroll_direction="down", scroll_amount=8, tabId=...)
   ```
2. **Wait 3 seconds** for new posts to render
3. **Take a screenshot** to see newly loaded posts
4. **Run `find` again** to extract the new posts
5. **Repeat** until you've collected enough posts or no new content loads
6. Typically 3-4 scroll cycles loads ~10-15 posts. For FULL_ANALYSIS (20 posts), do 5-6 cycles.

**De-duplication**: Posts may repeat across scroll loads. Track seen author+text combinations to avoid duplicates.

### Step 4: Convert Posts to Leads

For each relevant hiring post:

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
   - `path_strength`: "moderate" (post-based outreach converts well)
   - `outreach_status`: "pending"
   - `source`: "linkedin_post"
   - `profile_url`: extracted from the accessibility tree (`/in/{username}`)
   - `linked_roles`: [the role from the post]

### Step 5: Rate Limiting

Content search is slightly less throttled than job search, but still be careful:
- **Wait 3-4 seconds between search navigations**
- **Maximum 5-8 content searches per session**
- **Scroll slowly** — 3 seconds between scroll actions
- Stop on any CAPTCHA or unusual activity warnings
- A good session target: 3-5 search terms × 10-15 posts each = 30-75 posts scanned

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
