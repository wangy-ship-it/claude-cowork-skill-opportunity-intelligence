## PHASE 3.5: URL Pre-Triage ← NEW in v16

This phase exists because Runs 3, 4, and 28 all wasted significant time scoring and evaluating roles whose URLs turned out to be dead. Run 28 had 10 dead LinkedIn URLs that the user had to manually delete. Run 3 had an 83% URL liveness failure rate. Catching dead URLs early — before reading JDs in Phase 4 — saves the entire downstream pipeline from wasting cycles on ghost listings.

### Step 1: LinkedIn Job ID age check (instant, no network calls)

LinkedIn `/jobs/view/{ID}` URLs have roughly sequential IDs. As a quick heuristic:
- IDs below **3,500,000,000** are almost certainly expired (pre-2023) → **auto-drop**
- IDs between 3,500,000,000 and 4,000,000,000 are likely 2023-early 2024 → **flag as high-risk**
- IDs above 4,000,000,000 are likely recent → **proceed**

Apply this filter to all LinkedIn URLs from Channels 2 and 4. This is a heuristic, not a guarantee — but it catches the most obvious stale listings instantly.

### Step 2: Career site URL preference

For every role that has BOTH a LinkedIn URL and a company career site URL (or where the career site URL can be easily derived), **prefer the career site URL**. Career site URLs are far more stable — they don't expire when LinkedIn listings close. If a role was found via Channel 2 (LinkedIn) but the company is in the target list with a known career site, do a quick web search for `site:{company-careers-domain} "{exact job title}"` to find the direct career site URL.

### Step 3: Pattern-based URL rejection

Auto-drop any URL matching these patterns (these are search/browse pages, not specific job postings):
- Contains `?keywords=`, `?search_input=`, `?q=`, or `?query=`
- Is a bare board URL with no job ID (e.g., `boards.greenhouse.io/{company}` without `/jobs/{id}`)
- Is a LinkedIn search page (e.g., `linkedin.com/jobs/data-science-manager-jobs`)
- Redirects to a generic company careers landing page

### Step 3.5: Known-problematic URL sources ← NEW in v16.4

Some ATS board URLs look structurally valid (they have a job ID) but are known to redirect to generic search pages or return 403/404. These patterns have been confirmed across multiple runs. Rather than wasting Phase 7.5 verification cycles on them, flag them early.

**Known-problematic patterns (auto-flag as `URL_UNVERIFIED`):**
- **Anthropic Greenhouse** (`job-boards.greenhouse.io/anthropic/jobs/{id}`): Systematically redirects to a filtered search results page instead of the specific JD. The URL looks valid but never loads the actual posting. Flag as `URL_UNVERIFIED` and note "Anthropic GH redirects to search page" in Potential Gaps.
- **CareerPuck / JS-rendered boards** (`app.careerpuck.com/job-board/{company}/job/{id}`): Requires full JavaScript rendering to display content. WebFetch and simple HTTP checks will see an empty page. Flag as `URL_UNVERIFIED` and note "JS-required board, URL may not resolve" in Potential Gaps.
- **Chime Greenhouse** (`job-boards.greenhouse.io/chime/jobs/{id}`): Known to redirect to generic Chime careers page (not the specific role). Same handling as Anthropic.

**How to handle `URL_UNVERIFIED` roles:**
- Do NOT drop them — they may still be real, active roles
- Include them in output with the `URL_UNVERIFIED` flag
- In Phase 7.5, skip full browser verification for these (the known issue is already documented)
- In the CSV/Sheet, append "(URL unverified)" to Potential Gaps so the user knows the link may not work
- The user can manually navigate to the company's careers page and search for the role title

**Growing this list:** When a new company's Greenhouse/Lever/Ashby URLs are discovered to systematically fail in Phase 7.5, add the pattern here in a preferences update (Phase 10) so future runs catch it in pre-triage.

### Step 4: Print triage results

```
URL PRE-TRIAGE:
  Input: {X} roles from Phase 3
  LinkedIn ID age filter: {X} dropped (expired IDs)
  URL swapped to career site: {X}
  Pattern rejection: {X} dropped (search/browse pages)
  Known-problematic flagged: {X} as URL_UNVERIFIED (still included but flagged)
  Surviving: {X} roles → proceeding to Phase 3.7
```

This gate is lightweight (no browser navigation needed) and typically eliminates 20-40% of false positives before the expensive JD-reading phase.

---

## PHASE 3.6: Early URL Liveness Sampling ← NEW in v16.6

**Why this exists**: Run 11 invested full JD reads, signal extraction, and scoring on 13 roles — then Phase 7.5 dropped 12 of them (85.7% failure rate). The pipeline wasted most of its context budget on ghost listings. This lightweight sampling step catches catastrophic URL failures BEFORE investing in expensive downstream phases.

**This is NOT a replacement for Phase 7.5.** Phase 7.5 still does full verification with browser navigation. This phase does quick, cheap checks to catch obvious failures early and skip companies whose career sites are down or restructured.

### When to run this phase

- **ALWAYS run** when `market_health.url_404_rate` from the previous run was > 0.3 (30%+)
- **ALWAYS run** when any company in the queue has `company_intelligence.status` of `partial_freeze` or `hiring_freeze`
- **Skip** when previous run had url_404_rate < 0.15 AND no freeze/migration signals. The overhead isn't worth it when URLs are generally healthy.

### How it works: Stratified URL sampling

Rather than checking every URL (that's Phase 7.5's job), sample a subset to detect systemic failures early:

**Step 1: Group URLs by company**
```
Company A: 4 URLs
Company B: 3 URLs
Company C: 2 URLs
Company D: 1 URL
```

**Step 2: Sample 1 URL per company** (pick the one most likely to be valid — career site URL preferred over LinkedIn)

**Step 3: Quick liveness check** — For each sampled URL:
- **If browser available**: Navigate to URL, wait 3 seconds, check if page title/content suggests a real job posting or a 404/redirect/generic page. Take a screenshot for verification.
- **If no browser**: Use `WebFetch` on the URL and check for 404, redirect chains, or generic page content.
- **Time budget**: ~3 seconds per URL (navigation + check). Total for 5-8 companies = 15-25 seconds.

**Step 4: Assess results and decide**

For each company's sample:
- **LIVE** → All URLs from this company proceed to Phase 3.7 normally
- **DEAD (404/redirect/generic page)** → **Flag ALL URLs from this company** as `EARLY_DEAD`. These still proceed but are deprioritized:
  - In Phase 4 (JD reading): Read `EARLY_DEAD` URLs LAST (after all healthy URLs)
  - If context budget is tight, skip JD reads for `EARLY_DEAD` roles entirely and go straight to Phase 7.5 for full verification
  - This saves 60-80% of context when a company's entire career site is down (like DoorDash in Run 11)

**Step 5: Platform migration quick-detect**

During the sample check, also watch for these signals that a company has changed its career site:
- URL redirects to a completely different domain (e.g., `jobs.careers.microsoft.com` → `apply.careers.microsoft.com`)
- Page loads but job ID format has changed (old ID returns generic search)
- Career site shows a "we've moved" or "new careers site" banner

If detected, log a `PLATFORM_MIGRATION` flag for the company. This feeds into Phase 10's migration tracking (see below).

**Step 6: Print sampling results**

```
EARLY URL LIVENESS SAMPLING (Phase 3.6):
  Sampled: {X} URLs (1 per company, {Y} companies)
  Healthy: {list of companies} ✓
  Early dead: {list of companies} ✗ — all URLs from these companies flagged EARLY_DEAD
  Migration detected: {list if any}
  Decision: {e.g., "3 companies healthy, 1 flagged — proceeding with deprioritized reads for flagged"}
  Estimated context saved: {X} JD reads deferred
```

### Integration with Phase 7.5

Phase 7.5 still runs in full. The difference is:
- **Without Phase 3.6**: Phase 7.5 finds 85% dead URLs after the pipeline already invested in JD reads + scoring for all of them
- **With Phase 3.6**: The pipeline knows which companies are likely dead BEFORE JD reads. It can skip or defer those reads, then Phase 7.5 confirms the final set

---

## PHASE 3.7: Cross-Run Dedup Gate ← NEW in v16.5

**Why this moved here (from Phase 7.5)**: In v16.4, dedup happened AFTER scoring (Phase 7.5), which meant the pipeline read JDs, extracted signals, scored, and generated Action Packs for roles that were already in the sheet — wasting 40-60% of context window on duplicate work. Moving dedup to Phase 3.7 (after URL pre-triage, before JD reading) ensures we only invest context in confirmed-new roles.

Before reading any JDs, check every role from Phase 3.5 against ALL existing rows in the Google Sheet.

**How to dedup:**

1. Read the Google Sheet via Apps Script to extract all existing rows' Company (col G), Job Title (col D), and URL (col R).
2. For each role in this run's candidate list, check:
   - **Exact URL match**: Same URL already in sheet → **auto-drop**
   - **Company + Title match**: Normalize both (lowercase, strip whitespace/punctuation). If both match an existing row → **auto-drop**
   - **Fuzzy Title match**: Same company + title has >80% string overlap (e.g., "Sr Manager, Data Science" vs "Senior Manager of Data Science") → **auto-drop** with log
3. Log every dropped role: `"DEDUP: {Company} — {Title} already in sheet (row {X}, from {date})"`

**Display the dedup results:**
```
CROSS-RUN DEDUP GATE (Phase 3.7):
  Checked {X} roles against {Y} existing sheet rows
  Dropped: {N} duplicates
    - {Company} — {Title} (matched row {R}, {date}) [url_match/title_match/fuzzy_match]
  Passed: {M} unique new roles → proceeding to Phase 4 (JD reading)
```

If ALL roles are duplicates, skip Phases 4-9 and report "No new unique roles this run."

**Context savings**: This gate typically drops 40-80% of candidates before Phase 4, dramatically reducing the number of JDs that need to be read, scored, and Action-Packed. In Run 8, this would have eliminated 11 of 19 roles before any JD reading.

---

