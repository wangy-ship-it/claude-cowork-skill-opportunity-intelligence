## PHASE 3: Multi-Channel Search

Use **keyword matrix from Phase 2B**. Read `references/career-sites.md` for tips.

### Browser Detection (do this FIRST)

**Chrome browser check (MANDATORY — do this FIRST):** Call `tabs_context_mcp` to check if Chrome browser tools are available (`navigate`, `computer`, `read_page`, `javascript_tool`, `find`, `tabs_context_mcp`, `tabs_create_mcp`). Chrome is the **default execution method for Channel 1** (career sites) and the **strongly preferred method for Channels 2 and 4** (LinkedIn). If Chrome is available, create a tab group (`createIfEmpty: true`) and create tabs for browsing. Then read `references/linkedin-browser.md` for the Ch2/Ch4 browser workflows. If Chrome is NOT available, Ch1 falls back to web search (with major coverage loss), Ch2 degrades to web search, and Ch4 must be skipped entirely — log all of this in the Coverage Report.

**Channel 1 — Career Sites (MANDATORY — never skip)**:

Career sites are the **primary source of truth** for job discovery. Roles appear here first, data is fresher, and there is far less noise than aggregated boards like LinkedIn. This channel covers companies with **proprietary career portals** that are NOT accessible via Ch3 ATS APIs — meaning Ch1 is the only way to discover roles at Apple, Google, Meta, Microsoft, Netflix, Amazon, and Workday-based companies. Channel 1 MUST be attempted every run.

**⚡ Chrome browser (Claude in Chrome MCP) is the DEFAULT and PREFERRED method for Channel 1.** The Chrome MCP tools drive the user's actual Chrome browser, which can load any career site without restrictions. This is slower than API calls but produces the freshest, most complete results.

**Chrome MCP setup (do this ONCE at the start of Phase 3):**
1. Call `tabs_context_mcp` to check for an existing Chrome tab group. If none exists, call `tabs_context_mcp` with `createIfEmpty: true`.
2. Use `tabs_create_mcp` to create a dedicated tab for Ch1 career site browsing.
3. Store the tab ID — reuse this same tab for all Ch1 sites (navigate between them to save resources).

**Chrome MCP tools for Ch1:**
- `navigate` — load each career site URL
- `find` — locate search boxes, job listing elements, pagination controls
- `form_input` — enter search keywords into career site search bars
- `computer` (action: `left_click`) — click search buttons, filters, pagination links
- `computer` (action: `scroll`) — scroll through results pages
- `computer` (action: `screenshot`) — capture results for visual verification
- `read_page` — extract job titles, teams, locations, URLs from listing elements
- `get_page_text` — extract full text when structured element reading is difficult

**Fallback (ONLY if Chrome is genuinely unavailable):** If `tabs_context_mcp` fails or returns no response (Chrome extension not running), fall back to agent-based web search: `site:{company-careers-domain} "{job title}"`. This fallback has ~80%+ URL attrition and is dramatically inferior — log it as a coverage gap in the Coverage Report. Never silently skip Ch1; always attempt Chrome first.

**Target list — built dynamically each run (in priority order):**

1. **Apple (ALWAYS FIRST)**: Always start with `https://jobs.apple.com/en-us/search`. Apple is the anchor company for every run. Search all matrix queries.
2. **Interview-history companies**: Check `learning_history.conversion_rates.by_company` in preferences. Any company where `interviews >= 1` gets priority searching. These are proven warm targets — if they interviewed you before, they may have new matching roles.
3. **Warm-lead companies**: Check the Google Sheet tracking for companies where you have active referrals, networking contacts, or pending follow-ups. New roles at these companies are high-value because you already have an in.
4. **Major proprietary career sites**: Google (`careers.google.com`), Amazon (`amazon.jobs`), Meta (`metacareers.com`), Microsoft (`careers.microsoft.com`), Salesforce, Netflix (`jobs.netflix.com`). These use their own career portals, NOT third-party ATS.
5. **Workday companies** (from ATS registry "Not on any API" list): DoorDash, Snap, Adobe, Intuit, Capital One, Snowflake, OpenAI, Rippling, CrowdStrike. Workday sites must be searched via browser.
6. **New companies of interest**: Any company the user mentions or that appeared in Ch4 recruiter posts.

**Scope by mode:**
- FAST_TRACK: Apple + top 2 interview-history companies (3 sites total)
- FULL_ANALYSIS: Apple + all interview-history + all Tier 4-5 companies (8-12 sites total)

**Browser workflow per site:**
1. Navigate to the company's career/jobs page
2. **Check for company keyword override** (v16.7): If `company_intelligence.{company}.title_vocabulary.high_yield` exists, use those keywords FIRST. Otherwise, use the default matrix (start with tool-based queries — e.g., "data science manager", "analytics lead")
3. Filter by location if the site supports it (SF Bay Area, Remote)
4. Scroll/paginate through ALL results (don't stop at page 1)
5. Extract role title, team, location, and URL for each match
6. **Record result count per keyword** (v16.7): Log `{company}: "{keyword}" → {N} results` — this feeds the adaptive expansion logic below
7. Add to raw roles list with `source: "ch1_career_site"`
8. Move to next keyword, then next company
9. Wait 3-5 seconds between page loads to avoid rate limiting

### Adaptive Keyword Expansion ← NEW in v16.7

**Why this exists**: Run 12's DoorDash diagnostic showed that searching "data science" returned 1 result (Brazil only) while "analytics" returned 13 relevant roles — including all 3 that the user found manually. The pipeline reported "0 new roles found" because it never tried the right keyword. Career sites use strict keyword matching unlike LinkedIn's fuzzy search, so the exact term matters enormously.

**The rule is simple: when your primary keyword returns near-zero results on a career site, automatically try fallback keywords before moving on.**

**Trigger**: After Step 6 above, if primary keyword returned **≤ 2 results** at a company that was expected to have openings (not a confirmed `hiring_freeze`):

1. **Try the fallback keyword set** (in order): `"analytics"`, `"insights"`, `"strategy"`, `"data"`, `"machine learning"`
   - Skip any fallback that's already a primary keyword
   - Stop after finding a keyword with ≥ 5 results (no need to exhaust the list)
2. **Record what worked**: Log the high-yield keyword for this company
3. **Build vocabulary discovery**: If a fallback keyword returns significantly more results (10x+) than the primary keyword, this is a strong signal of vocabulary mismatch. Log it as `title_vocabulary` learning for Phase 10:
   ```
   KEYWORD EXPANSION TRIGGERED:
     {Company}: "{primary}" → {N} results (near-zero)
     Trying fallbacks: "analytics" → {X} results ← HIGH YIELD
     → Flagged for vocabulary update: high_yield += ["analytics"]
   ```

**Phase 10 integration**: Any vocabulary discoveries from adaptive expansion get written to `company_intelligence.{company}.title_vocabulary` in Phase 10, so the NEXT run uses the proven keywords from the start (no expansion needed).

**Context budget awareness**: Adaptive expansion adds 2-4 extra searches per company (10-20 seconds). Only trigger for companies where the primary keyword underperformed expectations. Don't expand for confirmed `hiring_freeze` companies.

**Important — ATS mapping**: Major tech companies (Microsoft, Google, Apple, Amazon, Meta, Salesforce, Netflix) use their **own proprietary career sites**, NOT third-party ATS platforms. Channel 1 is the right place to search these companies — don't duplicate effort by also searching Greenhouse/Lever for them in Channel 3. Channel 3 is for mid-size companies and startups that use third-party ATS.

**Channel 2 — LinkedIn Jobs** (use BOTH title-based AND tool-based queries from Phase 2B):

**⚡ Chrome browser (Claude in Chrome MCP) is the DEFAULT for Channel 2.** LinkedIn only exposes a small fraction of listings to web crawlers — browser access unlocks the full inventory. Web search fallback yields ~2 surviving roles vs 20-50+ with Chrome. The difference is dramatic and well-documented across 14+ runs.

**Chrome is the primary path.** Reuse the Chrome tab group established in Ch1 setup (or create one now via `tabs_context_mcp` if Ch1 didn't run yet). Create a new tab for LinkedIn browsing — keep it separate from the Ch1 career site tab.

**Chrome workflow:**
1. Follow `references/linkedin-browser.md` Channel 2 workflow
2. **Start with Freshness Sweep (Step 0, v16.5)** using `f_TPR=r3600` before standard search
3. Use `navigate` to load LinkedIn Jobs search URLs directly
4. Use `read_page` and `find` to extract job cards from results
5. Use `computer` (scroll, left_click) for pagination and filter interaction
6. Use `get_page_text` or `javascript_tool` to read promising JDs
7. Interleave title-based and tool-based queries for maximum coverage
8. Respect rate limits: 3-5 sec waits between pages, max 5 queries/session, stop immediately on CAPTCHA

**Fallback (ONLY if Chrome is genuinely unavailable):** Web search `site:linkedin.com/jobs "{query}"`. Use **tool-based queries ONLY** — title-based queries via web search have 0% survival rate (confirmed across multiple runs). Tool-based queries with level qualifiers achieve ~67% survival. Expect only 2-5 surviving roles total. Print `⚠ Ch2 operating in DEGRADED MODE (web search only) — estimated 80-90% of LinkedIn listings invisible` in the channel log and flag as a coverage blind spot in the Phase 8.5 Coverage Report.

**Scope:** FAST_TRACK: 3 queries / 2 pages each. FULL_ANALYSIS: all queries / 3 pages each.

**Channel 3 — Adaptive ATS Board Search** (Greenhouse, Ashby, Lever, SmartRecruiters):

This channel is most valuable for **mid-size companies and startups** that use third-party ATS platforms. Don't waste queries searching for companies that use their own career sites (already covered by Channel 1).

### The ATS Registry (stored in preferences)

Channel 3 is powered by an **ATS Registry** — a mapping of companies to their ATS platform and API access details. This registry grows over time as the system discovers new companies. It lives in `preferences.ats_registry` and has this structure:

```json
{
  "ats_registry": {
    "{company_name}": {
      "ats_type": "greenhouse|ashby|lever|smartrecruiters|workday|custom|unknown",
      "api_slug": "{slug_for_api_calls}",
      "api_works": true,
      "last_probed": "3/12/2026",
      "last_queried": "3/12/2026",
      "total_roles_last": 200,
      "filtered_roles_last": 3,
      "notes": "optional notes"
    }
  }
}
```

**If `ats_registry` doesn't exist in preferences**, initialize it with the seed data below on the first run, then save it in Phase 10.

### Four ATS APIs (all public, no auth required)

The system can query **four** ATS platforms via structured JSON APIs. Each call takes ~5 seconds and returns complete job listing data. Always apply **mandatory title filtering** after every API call (see filtering rules below).

**1. Greenhouse API** — Most common ATS for tech startups
```
https://boards-api.greenhouse.io/v1/boards/{slug}/jobs
```
Returns JSON with `id`, `title`, `location.name`, `updated_at`, `absolute_url` per job.

**2. Ashby API** — Growing ATS, excellent structured data
```
https://api.ashbyhq.com/posting-api/job-board/{slug}
```
Returns JSON with `jobs[]` array containing `id`, `title`, `department`, `team`, `location`, `employmentType`, `publishedAt`, `jobUrl`, `applyUrl`, and sometimes `compensation`.

**3. Lever API** — Requires `?mode=json` parameter
```
https://api.lever.co/v0/postings/{slug}?mode=json
```
Returns JSON array directly (no wrapper object). Each job has `id`, `text` (title), `categories.location`, `categories.team`, `hostedUrl`, `applyUrl`, `createdAt`. Also includes full JD as `descriptionPlain`.

**4. SmartRecruiters API** — Enterprise ATS with pagination
```
https://api.smartrecruiters.com/v1/companies/{CompanyName}/postings
```
Returns JSON with `totalFound`, `content[]` array. Each job has `id`, `name` (title), `location` (with city, region, remote flag), `department`, `experienceLevel`, `releasedDate`. Supports `?offset=X&limit=100` for pagination.

### Ch3 Data Limitations ← NEW in v16.4

ATS APIs return structured metadata, but some fields are often missing or unreliable compared to Ch1/Ch2:

- **Salary range**: Greenhouse and Lever APIs almost never include compensation data. SmartRecruiters sometimes includes `experienceLevel` but not salary. Only Ashby occasionally returns `compensation`. When salary is unavailable from the API, record as "N/A" in the CSV — do not guess or estimate. This is a known limitation, not a scoring failure.
- **Posted date**: Greenhouse returns `updated_at` (not `created_at`), which may reflect edits rather than original posting. Lever returns `createdAt` which is reliable. SmartRecruiters returns `releasedDate`. Ashby returns `publishedAt`. When the API field is missing or ambiguous, record as "N/A" and note "Ch3 — posting date unavailable" rather than leaving the field blank.
- **JD completeness**: Ch3 roles often have less context than Ch1/Ch2 because we're working from API metadata + a JD page that may or may not load (see Phase 3.5 known-problematic URLs). Score conservatively when JD is partially available.

These are inherent trade-offs of Ch3's speed advantage (querying 20+ companies in seconds vs. manual browser navigation). The user should expect more N/A values in Ch3-sourced rows.

### Mandatory Title Filtering (applies to ALL four APIs)

Every API returns ALL open roles at a company (often 100-500+). After fetching, keep only roles where the title matches BOTH:
1. A **domain keyword**: "data", "analytics", "science", "ML", "machine learning", "experimentation", "insights", "intelligence", "decision"
2. A **level keyword**: "manager", "lead", "head", "director", "senior manager", "principal", "VP"

Discard everything else. Report raw vs filtered counts per company:
```
{ATS_TYPE} API: {company} — {raw} total → {filtered} relevant after title filter
```

### Seed Registry (verified March 2026)

**Greenhouse** (20 companies):

| Company | Slug | Notes |
|---------|------|-------|
| Anthropic | `anthropic` | AI safety |
| Stripe | `stripe` | Fintech — filter aggressively |
| Instacart | `instacart` | Marketplace |
| Discord | `discord` | Community/gaming |
| Figma | `figma` | Design tools |
| Cloudflare | `cloudflare` | Large board (500+) |
| Datadog | `datadog` | Observability |
| Brex | `brex` | Fintech |
| Coinbase | `coinbase` | Crypto |
| Reddit | `reddit` | Social/content |
| Gusto | `gusto` | HR/payroll |
| Robinhood | `robinhood` | Fintech |
| Thumbtack | `thumbtack` | Marketplace |
| Calendly | `calendly` | Scheduling |
| Opendoor | `opendoor` | Real estate |
| Lyft | `lyft` | Mobility |
| Pinterest | `pinterest` | Visual discovery |
| Airbnb | `airbnb` | Travel |
| Headway | `headway` | Healthcare |
| GitLab | `gitlab` | DevOps |
| Databricks | `databricks` | Data/AI platform (785 roles — filter aggressively) |
| Toast | `toast` | Restaurant tech |
| Affirm | `affirm` | BNPL fintech |
| Chime | `chime` | Neobank |

**Ashby** (4 companies):

| Company | Slug | Notes |
|---------|------|-------|
| Notion | `notion` | Productivity — was "custom" in v16.1 |
| Ramp | `ramp` | Fintech — was "custom" in v16.1 |
| Plaid | `plaid` | Fintech (also on Lever) |
| NerdWallet | `nerdwallet` | Personal finance |

**Lever** (4 companies):

| Company | Slug | Notes |
|---------|------|-------|
| Palantir | `palantir` | Defense/enterprise (238 roles) |
| Wealthfront | `wealthfront` | Fintech |
| Spotify | `spotify` | Music/streaming (153 roles) |
| Plaid | `plaid` | Also on Ashby — prefer Ashby (structured) |

**SmartRecruiters** (1 company — probe for more):

| Company | Slug | Notes |
|---------|------|-------|
| Visa | `Visa` | Large enterprise (1174 roles) |
| Uber | `uber` | Rideshare (small board — 1 role found) |

**Not on any API** (use career sites / site: search): DoorDash, Rippling, Snap (Workday), Adobe (Workday), Intuit (Workday), Capital One (Workday), CrowdStrike, Snowflake, OpenAI.

### ATS Probe Protocol (for new companies)

When a **new company enters the search queue** (via Tier 3 discovery or user addition) and is NOT in the ATS registry, run the **ATS Probe** to determine its ATS type. This takes ~20 seconds and only needs to run once per company.

**Probe order** (try each, stop at first success):

```
STEP 1: Greenhouse → WebFetch https://boards-api.greenhouse.io/v1/boards/{slug}/jobs
  If JSON with jobs[] → ats_type = "greenhouse", save slug

STEP 2: Ashby → WebFetch https://api.ashbyhq.com/posting-api/job-board/{slug}
  If JSON with jobs[] → ats_type = "ashby", save slug

STEP 3: Lever → WebFetch https://api.lever.co/v0/postings/{slug}?mode=json
  If JSON array with id/text fields → ats_type = "lever", save slug

STEP 4: SmartRecruiters → WebFetch https://api.smartrecruiters.com/v1/companies/{CompanyName}/postings
  If JSON with content[] → ats_type = "smartrecruiters", save slug
  Note: SmartRecruiters uses PascalCase company names, not lowercase slugs

STEP 5: If all 4 fail → ats_type = "unknown", api_works = false
  Fall back to site: searches or Channel 1 career site
```

**Slug guessing**: Try lowercase company name first, then common variations:
- `doordash`, `door-dash`, `DoorDash`
- `crowdstrike`, `crowd-strike`, `CrowdStrike`
- For SmartRecruiters, try both `CompanyName` and `companyname`

**Print probe results:**
```
ATS PROBE: {company}
  Greenhouse: {✓ found ({X} roles) / ✗ 404}
  Ashby:      {✓ found ({X} roles) / ✗ 404}
  Lever:      {✓ found ({X} roles) / ✗ 404}
  SmartRecr:  {✓ found ({X} roles) / ✗ 404}
  → Result: {ats_type} (slug: {slug}) — saved to registry
```

### When to Probe

- **Tier 3 discovery**: Any new company found via keyword search → auto-probe before next run's Channel 3
- **User adds a company**: Probe immediately
- **Registry entry is stale** (>30 days since `last_probed`): Re-probe to catch ATS migrations
- **API call fails** for a previously working company: Re-probe (company may have migrated)
- **FULL_ANALYSIS expansion**: When Channel 3 has time budget remaining, probe the next 3-5 unprobed companies from `company_search_log`

### Channel 3 Execution Flow

```
1. Load ats_registry from preferences
2. Build query list: all companies in search queue that have api_works = true
3. For each company with a known working API:
   a. Call the appropriate API endpoint
   b. Apply mandatory title filtering
   c. Extract roles with direct URLs
4. For companies with ats_type = "unknown" or api_works = false:
   a. Fall back to site: Google searches (Lever, Ashby, SmartRecruiters)
   b. If a new company appeared in Tier 3 discovery → run ATS Probe
5. Cross-company dedup
6. Print Channel 3 summary
```

### Adaptive Mode Budget

**ALL modes MUST query ALL registry companies with `api_works = true`.** ATS API calls are fast (seconds each) and the registry exists precisely so we don't miss companies. Querying only a subset defeats the purpose — Run 14 queried 9 of 34 companies and missed potential matches at the other 25. The marginal cost of each additional API call is ~5 seconds; the cost of missing a good role is weeks of delayed job search.

- **FULL_ANALYSIS**: Query ALL + probe up to 5 new companies + run site: searches for unknowns.
- **FAST_TRACK**: Query ALL + probe up to 2 new companies.
- **QUICK_TRIAGE**: Query ALL (no probes, no site: searches).

### Channel 3 Summary (updated for multi-ATS)

**After all API calls, probes, and site: searches, print:**
```
CHANNEL 3 SUMMARY:
  ATS APIs queried:
    Greenhouse: {X} companies → {raw} raw → {filtered} after filter
    Ashby:      {X} companies → {raw} raw → {filtered} after filter
    Lever:      {X} companies → {raw} raw → {filtered} after filter
    SmartRecr:  {X} companies → {raw} raw → {filtered} after filter
  Site: searches: {X} executed → {site_total} roles
  New companies probed: {X} ({results summary})
  Aggregators flagged: {X} ({names if any})
  Registry size: {X} companies ({Y} with working APIs, {Z} unknown)
  TOTAL Channel 3: {grand_total} roles proceeding to Phase 3.5
```

### Learning Loop (Phase 10 update)

In Phase 10, update the ATS registry with this run's results:
```
PHASE 10 — ATS REGISTRY UPDATE:
[ ] Update last_queried for each company queried this run
[ ] Update total_roles_last and filtered_roles_last per company
[ ] Add newly probed companies with their ats_type and slug
[ ] Flag companies where API returned errors (set api_works = false, add note)
[ ] Re-probe any company with api_works = false that hasn't been probed in 30+ days
```

Over time, the registry grows automatically: Tier 3 discovery finds a new company → ATS Probe determines its platform → the company enters the registry → future runs query it via API. After 10-20 runs, the registry will cover most of the addressable market for your profile.

**Aggregator detection**: Some ATS boards belong to job aggregator companies (e.g., Jobgether, Crossover, Turing, Andela) that repost roles from other companies. These produce lower-quality leads because: the URL goes to the aggregator's board, not the actual company; the role details may be incomplete or altered; and applying through an aggregator adds an intermediary. When you see results from known aggregators, **flag them** with `[AGGREGATOR]` and deprioritize them below direct company postings. Still include them (they may be the only route to certain roles), but never rank them above direct postings.

**Channel 4 — LinkedIn Recruiter/HM Posts** (highest-value channel — warm leads with 3-5x response rates):

**⚡ Chrome browser (Claude in Chrome MCP) is REQUIRED for Channel 4.** There is no effective non-browser fallback — LinkedIn posts are not indexed by search engines, so web search returns 0 usable leads. Reuse the Chrome tab group from Ch1/Ch2 setup. If Chrome is unavailable, skip Ch4 entirely and log it as a coverage gap — do not waste time on web search fallback for this channel.

**Human-like LinkedIn browsing strategy** — Ch4 should mirror how a real person browses LinkedIn for hiring signals. Do NOT just jump to keyword search URLs mechanically. Follow this natural browsing flow:

**Step A — Home Feed Scan (do this FIRST):**
Navigate to `https://www.linkedin.com/feed/`. Scroll through your home feed (4-6 scroll cycles). Your connections' hiring posts surface here organically — these are the **warmest possible leads** because you already have a 1st-degree connection. Look for posts mentioning "hiring", "open role", "join my team", "growing the team", "we're looking for". Use `find` and `read_page` to extract. This takes 2-3 minutes and often catches posts that keyword search misses.

**Step B — Notifications Check:**
Navigate to `https://www.linkedin.com/notifications/`. Scan for: recruiters viewing your profile, hiring managers engaging with your content, connection requests from talent acquisition. These are inbound signals — someone is already interested. Extract any relevant leads.

**Step C — Targeted Content Search (keyword-based):**
Now do the structured keyword search from `references/linkedin-browser.md` Channel 4 workflow. Use content search URLs with hiring-related terms. **IMPORTANT**: LinkedIn's content search page uses obfuscated CSS and SVG rendering — JavaScript CSS selectors will NOT work. Use `find` and `read_page` (accessibility tree) to extract post authors, titles, and content.

Expand search term variety beyond the defaults. Include:
- Role-specific: `"hiring data science manager"`, `"open role analytics lead"`
- Action-oriented: `"we're hiring"`, `"join my team"`, `"growing my team"`, `"looking for"`, `"just opened"`
- Urgency signals: `"backfill"`, `"immediate hire"`, `"ASAP"`, `"starting search"`
- Team signals: `"building a team"`, `"new team"`, `"expanding"`

**Step D — Target Company Page Browsing:**
For your top 3-5 target companies (from Ch1 interview-history list), navigate to their LinkedIn company page → click "Posts" tab. Companies often announce new roles through their official LinkedIn posts before they hit the career page. This catches roles at the "just opened" stage.

**Step E — My Network / Job Changes:**
Navigate to `https://www.linkedin.com/mynetwork/`. Check "People you may know" and recent job changes in your network. Someone who just joined a target company is a warm referral source. Someone who just became a hiring manager is a warm lead. Extract relevant contacts for the CRM.

**Scope by mode:**
- FAST_TRACK: Steps A + C (3 search terms, 10 posts each)
- FULL_ANALYSIS: All steps A-E. Step C: all search terms, 20 posts each. Step D: top 5 companies.

Extract signals, convert to leads with +15 IP. Classify posters as HM/recruiter/team member, generate post-referencing outreach drafts, add contacts to CRM.

- **If no browser tools — DO NOT run web search fallback.** LinkedIn content/feed posts are NOT indexed by search engines. The `site:linkedin.com "hiring" "{query}"` approach returns only stale posts (often 2+ years old) and wastes 30-60 seconds of run time that would be better spent deepening Channels 1 and 3. Instead: mark Channel 4 as `⏭ SKIPPED (browser required)` in the completion log, and **reallocate the time budget** to additional Channel 1 pagination or Channel 3 Greenhouse API expansion (see expanded company list below).

**Reallocation priority when Ch4 is skipped**: Use the saved time budget to query additional Greenhouse API companies from the expanded list. Each API call takes ~5 seconds and can surface 1-5 relevant roles after filtering. This is a far better use of time than the 0 roles web search would produce.

If LinkedIn blocked (CAPTCHA, login wall, rate limit): log the specific error, report what was found so far, proceed with available channels. Never retry aggressively — the user's LinkedIn account is at stake.

**Cross-channel dedup** (4 layers): URL → normalized key → fuzzy title+location → job ID.

### ═══ CHANNEL COMPLETION GATE (BLOCKING) ═══

Phase 4 CANNOT start until this gate passes. The reason this gate exists is that historically, channels got silently dropped (especially Channel 4) and the user only discovered it after the run was complete. This wastes the user's time and misses potential warm-channel leads that are the most valuable outcomes.

Before moving to Phase 4, print this channel log and verify every channel was attempted:

```
CHANNEL COMPLETION LOG:
  Ch1 (Career Sites):         [✓/✗] — {X} roles found | {notes}
  Ch2 (LinkedIn Jobs):        [✓/✗/⚠] — {X} roles found | {browser/web_search} | title-based: {X}, tool-based: {X} | {notes}
  Ch3 (Boolean/ATS):          [✓/✗] — {X} roles found | API: {X}, site-search: {X} | {notes}
  Ch4 (Recruiter/HM Posts):   [✓/⏭/✗] — {X} roles found | {browser/skipped_no_browser} | {notes}
  Browser tools available:    [YES/NO]
  All channels attempted:     [YES/NO]
  Time reallocated from Ch4:  {where time went, if Ch4 skipped}
```

Note: ⚠ means the channel was attempted via web search fallback (browser unavailable). ⏭ means the channel was intentionally skipped because the fallback method is ineffective (Channel 4 without browser — LinkedIn posts are not indexed by search engines, so web search returns only stale 2+ year old posts).

**If any channel shows ✗**: Go back and run it before proceeding. A channel returning 0 results is fine (mark ✓ with "0 roles found") — the requirement is that the search was *attempted*, not that it produced results. The only acceptable exception is if a platform is completely blocked/down (e.g., LinkedIn returns a login wall), in which case log the specific error.

**Channel 4 skip rule**: If browser tools are NOT available, Channel 4 shows ⏭ (not ✗). This is acceptable because web search fallback for LinkedIn content posts is proven ineffective — it wastes time and produces misleading stale results. The time saved should be reallocated to deeper searching in Channels 1 and 3 (noted in the log).

**This applies to ALL modes** including QUICK_TRIAGE and FAST_TRACK. The scope per channel varies by mode (QUICK_TRIAGE does lighter searches), but Channels 1-3 must always be attempted. Channel 4 requires browser tools to function — without them, skip and reallocate.

---

