---
name: job-search
description: |
  **Personal CRM + Opportunity Intelligence System (v15)**: Three-domain platform — Opportunity Discovery (4-channel search, signal extraction, Fit/Opportunity/IP scoring, APPLY NOW/NETWORK FIRST/HOLD/SKIP gates), Relationship CRM (contact tracking, 8-state networking pipeline DISCOVERED→CLOSED, automated follow-ups, strength transitions), Interview Pipeline (stage tracking, prep tasks, thank-you notes). Discovery Mode (weekly) and Networking Mode (daily). Learns from cluster and channel performance.

  Trigger when user mentions "jobs", "careers", "openings", "roles", "positions", "hiring", "apply", "find roles matching my resume", "daily job search", "run my job search", "find referrals", "who should I network with", "follow up", "check my pipeline", "interview prep", "networking update", "any follow-ups due", or "run networking mode".
---

# Personal CRM + Opportunity Intelligence System v15.0 — Execution Guide

> **Design Philosophy**: This agent operates as a personal opportunity intelligence and networking management platform — not just a job discovery engine. It manages relationships, tracks interview pipelines, and discovers opportunities across three integrated domains.

You are running a **warm-first** job search and networking management system. Every run ends with the user knowing exactly what to do next. Each run gets smarter because the system learns from outcomes.

**Three core domains**:
1. **Opportunity Discovery** — Find and score roles across 4 channels
2. **Relationship / Networking CRM** — Track contacts, manage outreach pipelines, automate follow-ups
3. **Interview Pipeline Management** — Track interviews, generate prep, manage thank-you notes

**Core philosophy**: Warm channels (referrals, recruiter outreach, HM posts) convert at 80-100% vs cold at 0-5%. The primary output is **networking actions**, not just applications. Cold applications are deprioritized. Every role gets a referral-finding plan. Quality of opportunities pursued always beats quantity of applications submitted.

**Two operational modes**:
- **Discovery Mode** — Runs occasionally (e.g., weekly). Full pipeline: search for new roles, score, evaluate networking paths. Use when the user wants to find new opportunities.
- **Networking Mode** — Runs frequently (e.g., daily). Focuses on CRM follow-ups, interview pipeline updates, and relationship management. Skips new role discovery. Use when the user wants to manage existing pipeline.
- The system should **dynamically prioritize Networking Mode** when discovery yield is low (market tightening, few new roles, many active pipelines).

## Reference Files

This SKILL.md is the execution backbone. Detailed rubrics and formats live in reference files — read them when you reach the relevant phase:

- `references/onboarding.md` — First-run checkpoints, deep candidate analysis, enhanced contextual matching
- `references/networking_crm.md` — **NEW in v14** — Contact record schema, networking pipeline states (DISCOVERED→CLOSED), follow-up automation rules, relationship strength transitions, CRM maintenance. Read before Phase 1.5 and Phase 5.5.
- `references/interview_pipeline.md` — **NEW in v14** — Interview stage tracking, prep task generation, thank-you note management, interview pipeline dashboard. Read before Phase 1.7.
- `references/signal_extraction.md` — Structured signal extraction from JDs (role, domain, technology, company signals). Read before Phase 4.5.
- `references/scoring.md` — Fit Score rubric, Opportunity Score, Interview Probability, Action Gates, adaptive thresholds, bucket quotas, opportunity clustering
- `references/networking_strategy.md` — Networking path strength classification (Strong/Moderate/Weak), outreach templates, referral search plans. Read before Phase 5.5.
- `references/decision_engine.md` — Reasoning requirements for every recommendation. Read before Phase 5.
- `references/guardrails.md` — Hard constraints preventing wasteful strategies. Read at pipeline start.
- `references/learning_loop.md` — Channel performance tracking, opportunity cluster learning. Read before Phase 1 and Phase 10.
- `references/output-formats.md` — CSV 26-column schema, Action Pack format, Summary Report template, Google Sheet write process, tailoring tiers, Relationship Table, Interview Pipeline Dashboard
- `references/career-sites.md` — Company-specific navigation tips, ATS patterns, LinkedIn/Boolean search templates
- `references/linkedin-browser.md` — **NEW in v15** — Chrome browser automation for LinkedIn Jobs (Channel 2) and Recruiter/HM Posts (Channel 4). Read before Phase 3 when browser tools are available.

---

## ═══ MANDATORY EXECUTION CHECKLIST ═══

This checklist is the beating heart of the pipeline. Every phase MUST produce a specific output artifact before the next phase begins. If a phase is skipped or its output is missing, the run is incomplete.

**Print this checklist at the START of every run. Check off each item as you complete it. At the end, verify all items are checked.**

**At pipeline start: Read `references/guardrails.md` NOW — it defines hard limits (max 3 apps/day, Fit floor, black hole prevention, warm-over-cold). These constraints override scoring outputs.**

```
RUN CHECKLIST — {date} — Mode: {DISCOVERY / NETWORKING} × {QUICK_TRIAGE / FAST_TRACK / FULL_ANALYSIS}

[ ] PHASE 0: Load preferences + CRM memory from {workspace}/job-search-preferences.json
    → OUTPUT: preferences object loaded (or trigger onboarding if missing)
    → OUTPUT: CRM contacts and opportunity pipelines loaded

[ ] PHASE 1: Daily Check-in (returning users) OR Onboarding (first run)
    → Read references/learning_loop.md for channel performance tracking
    → OUTPUT: Daily Briefing displayed to user (see template below)
    → OUTPUT: Updated learning_history in memory (including channel performance)

[ ] PHASE 1.5: Manage Networking Follow-ups (NEW in v14)
    → Read references/networking_crm.md first
    → OUTPUT: Follow-up dashboard displayed (overdue, due today, upcoming)
    → OUTPUT: Networking pipeline states updated for all active opportunities
    → OUTPUT: Relationship strength transitions applied (promotions/expirations)
    → OUTPUT: Follow-up actions recommended with specific templates

[ ] PHASE 1.7: Update Interview Pipeline (NEW in v14)
    → Read references/interview_pipeline.md first
    → OUTPUT: Interview pipeline dashboard displayed
    → OUTPUT: Prep tasks generated for upcoming interviews
    → OUTPUT: Overdue thank-you notes flagged
    → OUTPUT: Interview stage updates applied from sheet changes

--- DISCOVERY MODE PHASES (skip if Networking Mode only) ---

[ ] PHASE 2A: Build search queue
    → OUTPUT: Search queue displayed with Tier 1/2/3 breakdown (see template below)

[ ] PHASE 2B: Keyword Intelligence Engine
    → OUTPUT: Keyword matrix printed (proven + discovery keywords with confidence scores)
    → BLOCKING: Phase 3 cannot start without the keyword matrix

[ ] PHASE 2C: Market Health Check
    → OUTPUT: Market health assessment printed (url_404_rate, trend, responsive actions)
    → If tightening detected: lower NETWORK FIRST threshold (Fit ≥ 65), increase Tier 3 by 2, propose strategy shift

[ ] PHASE 2D: Company Intelligence
    → OUTPUT: Company health status for each company in queue

[ ] PHASE 3: Multi-Channel Search (ALL 4 channels required for ALL modes)
    → Channel 1 (Career Sites): [ ] completed — {X} roles found
    → Channel 2 (LinkedIn Jobs): [ ] completed — {X} roles found
    → Channel 3 (Boolean/ATS): [ ] completed — {X} roles found
    → Channel 4 (LinkedIn Recruiter/HM Posts): [ ] completed — {X} roles found
    → Cross-channel dedup: [ ] completed — {X} unique roles after dedup
    → CHANNEL COMPLETION GATE: [ ] All 4 channels attempted (print log before Phase 4)

[ ] PHASE 4: Read and evaluate JDs
    → OUTPUT: JD snapshots saved to {workspace}/jd/
    → OUTPUT: Stale posting filter applied — {X} auto-skipped, {X} warned

[ ] PHASE 4.5: Extract structured signals
    → Read references/signal_extraction.md first
    → OUTPUT: Signal block produced for each scoreable role
    → OUTPUT: Opportunity cluster tag assigned to each role
    → BLOCKING: Phase 5 cannot start without signal blocks

[ ] PHASE 5: Score all roles (read references/scoring.md first)
    → Scores consume extracted signals, NOT raw JD text
    → OUTPUT: Three scores per role (Fit, Opportunity, Interview Prob)
    → OUTPUT: Adaptive thresholds applied if needed
    → OUTPUT: If <5 APPLY NOW at floor → company expansion triggered: [ ] yes/no

--- END DISCOVERY-ONLY PHASES ---

[ ] PHASE 5.5: Evaluate networking path strength + update CRM
    → Read references/networking_strategy.md and references/networking_crm.md
    → OUTPUT: Path strength classified for each role (Strong / Moderate / Weak)
    → OUTPUT: Networking boost applied to Interview Probability
    → OUTPUT: Referral search plan generated for NETWORK FIRST and HOLD roles
    → OUTPUT: New contacts added to CRM, pipeline states updated

[ ] PHASE 6: Determine engagement strategy + generate Action Pack
    → Read references/decision_engine.md for reasoning requirements
    → Read references/guardrails.md for hard constraints
    → OUTPUT: Action gates assigned (APPLY NOW / NETWORK FIRST / HOLD / SKIP)
    → OUTPUT: Reasoning block for each role
    → 6A-C: Tailored resume bullets + outreach for APPLY NOW roles
    → 6D: Referral-finding plans for ALL NETWORK FIRST roles
    → 6E: HOLD roles get brief referral search queries and next action
    → OUTPUT: Action Pack saved to {workspace}/Job_Search_Action_Pack_{date}.md

[ ] PHASE 7: Generate CSV
    → OUTPUT: CSV saved with EXACTLY 26 columns (A-Z) per output-formats.md § Column-to-Array Position Mapping
    → VERIFY: Each row has 26 values; Location is at index [5], Company at [6], URL at [17]
    → URL verification pass completed

[ ] PHASE 8: Summary Report (use EXACT template from references/output-formats.md)
    → OUTPUT: Summary displayed with ALL sections
    → OUTPUT: Relationship Table displayed (contacts, statuses, follow-ups)
    → OUTPUT: Interview Pipeline Dashboard displayed

[ ] PHASE 9: Append to Google Sheet (read output-formats.md § Column-to-Array Mapping FIRST)
    → PRE-WRITE: Validate each data row has exactly 26 values in correct column order
    → PRE-WRITE: Confirm Location at [5], Company at [6], scores as integers at [9-11], URL at [17]
    → OUTPUT: {X} rows written, verified
    → POST-WRITE: Switch to sheet tab, screenshot last rows to confirm data + column alignment
    → FULL_ANALYSIS: Apply conditional formatting (see output-formats.md)

[ ] PHASE 10: Save preferences + update learning + update CRM + update interview pipeline
    → STOP: Read references/learning_loop.md NOW before updating anything
    → Print the Phase 10 Field Checklist (see Phase 10 section below)
    → OUTPUT: ALL fields updated — check off each one individually
    → KEY RULE: Update EXISTING company_intelligence keys, NEVER create duplicates
    → KEY RULE: Update CRM contacts and opportunity pipeline states
    → KEY RULE: Update interview pipeline entries
```

**ENFORCEMENT RULE**: Before starting Phase N+1, confirm Phase N's output exists. If you realize you skipped a phase, STOP and go back. Thoroughness beats speed.

---

## Preferences File

Location: `{workspace}/job-search-preferences.json`

At run start: check if file exists.
- **Exists** → load, proceed to Daily Check-in
- **Missing** → run onboarding (read `references/onboarding.md`)

Always save after successful run. Key fields: `last_run`, `run_counter`, `learning_history`, `keyword_wisdom`, `market_health`, `company_search_log`, `application_cooldowns`, `sourcing_performance`, `company_intelligence`, `referral_tracking`, `channel_performance`, `cluster_outcomes`, `networking_crm`, `opportunity_pipelines`, `interview_pipeline`.

The preferences schema includes:
- **Core**: `candidate_profile`, `location_preferences`, `target_companies`, `google_sheet_url`, `minimum_salary_base`, `search_channels`
- **Intelligence**: `company_intelligence`, `keyword_wisdom`, `application_cooldowns`, `market_health`
- **Learning**: `learning_history` (with conversion_rates, hot_patterns, cold_patterns, black_hole_companies, score_adjustments, weekly_kpis, strategy_changes, channel_performance, cluster_outcomes)
- **CRM (NEW in v14)**: `networking_crm` (with contacts[]), `opportunity_pipelines` (with per-opportunity networking state tracking) — see `references/networking_crm.md`
- **Interview Pipeline (NEW in v14)**: `interview_pipeline[]` (with per-opportunity interview stage, prep tasks, thank-you tracking) — see `references/interview_pipeline.md`
- **Performance**: `referral_tracking`, `sourcing_performance`, `channel_performance`, `cluster_outcomes`

---

## PHASE 1: Daily Check-in — The Learning Engine

**STOP: Read `references/learning_loop.md` NOW** before proceeding. It defines channel performance metrics, cluster outcome tracking, and the display format for the Daily Briefing. Without reading it first, the channel performance and cluster sections of the briefing get skipped or formatted incorrectly.

Runs BEFORE any searching. Turns the tracking sheet into a learning system.

### Step 1: Read sheet and diff against last_run
Open Google Sheet, read ALL rows. Identify since last run:
1. New Applied = TRUE rows
2. New Interview = TRUE rows (extract company, bucket, team type, JD keywords, scores)
3. Stage changes
4. Referral = TRUE rows
5. User-edited fields (respect manual changes as feedback)

### Step 2: Compute learning metrics
Update in preferences:
- **Conversion rates**: by_company, by_bucket, by_ats_type, overall
- **Hot patterns**: ≥2 interviews with common pattern → +5/+10/+15 boost (signal_count 2/3-4/5+)
- **Cold patterns**: ≥3 rejections → -5/-10/-15 penalty
- **Black holes**: ≥5 apps, 0 screens, 30+ days → cap IP at 55
- **Score adjustments**: keyword boosts/penalties, company modifiers
- **Sourcing performance**: by_channel rates. When warm >3x cold with ≥5 data → auto cold B penalty
- **Channel performance**: Update per-channel metrics (see learning_loop.md § Channel Performance Tracking)
- **Cluster outcomes**: Update per-cluster interview rates (see learning_loop.md § Opportunity Cluster Learning)
- **Referral tracking**: expire 30+ day unresponsive contacts
- **Weekly KPIs**: snapshot if 7+ days since last

### Step 3: Auto-adjust scoring
Silently apply and report changes.

### Step 4: Propose strategy changes (need user approval)
Triggers: drop black hole company, add new company type, shift bucket allocation, adjust level targeting, widen/narrow location, channel strategy shifts.

### Step 5: Show Daily Briefing ← MANDATORY OUTPUT

```
═══════════════════════════════════════════════════
DAILY CHECK-IN — {M/D/YYYY}
═══════════════════════════════════════════════════

SINCE LAST RUN ({last_run}):
  New applications: {X}
  New interviews: {X} ({list})
  Stage changes: {X}
  Pipeline: {X} active, {Y} pending

WHAT'S WORKING:    {hot pattern} — {X} interviews
WHAT'S NOT WORKING: {cold pattern} — {X} rejections, 0 interviews

CHANNEL PERFORMANCE:
  {channel}: {apps} apps → {interviews} interviews ({rate}%) {★ if best / ⚠ if worst}
  ...
  INSIGHT: {key takeaway}

BEST-PERFORMING CLUSTERS: {cluster1} ({rate}%), {cluster2} ({rate}%)

SCORING ADJUSTMENTS: +{X} for {kw} | -{X} for {kw} | {Company} black hole

WEEKLY KPIs: Apps: {X} | Screens: {X} | Conversion: {X}% | Trend: {direction}

{STRATEGY PROPOSALS if any}
Ready to search. Any changes?
```

---

## PHASE 1.5: Manage Networking Follow-ups ← NEW in v14

**STOP: Read `references/networking_crm.md` NOW.** It defines the contact record schema, networking pipeline states, follow-up rules, and relationship strength transitions. Without reading it first, follow-up logic and state transitions will be inconsistent.

This phase runs EVERY run (both Discovery and Networking modes). It's the heart of the relationship management system.

### Step 1: Load CRM data
Load `preferences.networking_crm.contacts` and `preferences.opportunity_pipelines`.

### Step 2: Check for overdue follow-ups
For each contact with `outreach_status` = `message_sent`:
- Calculate days since `last_contact_date`
- If 5-7 days with no response → add to follow-up queue with template
- If 14+ days with no response → mark `no_response`, downgrade relationship strength

### Step 3: Check referral pipeline
For each contact with `outreach_status` = `referral_offered`:
- Check linked opportunities — has application been submitted?
- If not → prompt immediate application ("You have a referral waiting — apply now!")

For each contact with `outreach_status` = `referral_submitted`:
- Check if interview has been received (from sheet)
- If 14+ days with no interview → suggest check-in message

### Step 4: Apply relationship strength transitions
- Contacts who responded positively → elevate strength (weak→moderate)
- Contacts unresponsive 30+ days → downgrade strength
- Contacts who submitted referrals → mark as strong

### Step 5: Update networking pipeline states
For each active opportunity in `opportunity_pipelines`:
- Sync state with sheet data (Applied, Interview checkboxes)
- Advance state if conditions met (e.g., Applied=TRUE → APPLICATION_SUBMITTED)

### Step 6: Display Follow-up Dashboard ← MANDATORY OUTPUT

```
═══════════════════════════════════════════════════
NETWORKING FOLLOW-UPS DUE
═══════════════════════════════════════════════════

OVERDUE:
  ⚠ {name} @ {company} — outreach sent {date}, no response ({X} days)
    → ACTION: Send follow-up message

DUE TODAY:
  📬 {name} @ {company} — {status} {date}
    → ACTION: {specific next step}

UPCOMING (next 3 days):
  📅 {name} @ {company} — {action} due {date}

RECENTLY COMPLETED:
  ✓ {name} @ {company} — {outcome} {date}

PIPELINE SUMMARY:
  Active contacts: {X} | Overdue: {X} | Referrals pending: {X}
  States: {X} OUTREACH_SENT, {X} RESPONSE_RECEIVED, {X} REFERRAL_SECURED
```

---

## PHASE 1.7: Update Interview Pipeline ← NEW in v14

**STOP: Read `references/interview_pipeline.md` NOW.** It defines interview stage tracking, prep task generation, and thank-you note management.

This phase runs EVERY run (both Discovery and Networking modes).

### Step 1: Load interview pipeline
Load `preferences.interview_pipeline[]`.

### Step 2: Sync with sheet data
Read Google Sheet for stage changes, new Interview=TRUE rows. Update interview pipeline entries to match.

### Step 3: Generate prep tasks
For interviews scheduled within the next 7 days:
- Generate stage-appropriate prep tasks (phone screen, technical, onsite)
- Check completion status of existing prep tasks

### Step 4: Check thank-you notes
Flag any completed interviews where thank-you notes haven't been sent.

### Step 5: Generate reminders
- Interview in 2 days → "Prep reminder: {X}/{Y} tasks done"
- Interview tomorrow → "Final prep checklist"
- Post-interview, no update in 7 days → "Consider following up"

### Step 6: Display Interview Pipeline Dashboard ← MANDATORY OUTPUT

```
═══════════════════════════════════════════════════
INTERVIEW PIPELINE — {date}
═══════════════════════════════════════════════════

ACTIVE INTERVIEWS:
  🔵 {Company} — {Role}
     Stage: {stage} ({date})
     Prep: {X}/{Y} tasks done | Thank-you: {status}
     Next: {action}

PENDING RESPONSE:
  ⏳ {Company} — {Role}
     Applied: {date} | Days waiting: {X}

RECENTLY CLOSED:
  {✓/✗} {Company} — {Role} — {outcome} ({date})

PIPELINE HEALTH:
  Active: {X} | Pending: {X} | Win rate: {X}/{Y} ({%})
```

---

## PHASE 2A: Build Search Queue ← MANDATORY OUTPUT

**Tier 1**: Companies with interviews/referrals/active pipeline. Search new postings since last_run.
**Tier 2**: Remaining target_companies by rotation. FAST_TRACK: max 3/day. FULL_ANALYSIS: all.
**Tier 3**: Discovery. FAST_TRACK: 1-2. FULL_ANALYSIS: 3-5. Discover via hot_patterns, competitors, job boards.

```
TODAY'S SEARCH QUEUE:
  Tier 1: {companies}
  Tier 2: {companies} (not searched in {X}+ days)
  Tier 3: {companies} (new — via {reason})
  Total: {X} companies | Mode: {mode}
```

---

## PHASE 2B: Keyword Intelligence Engine ← BLOCKING GATE

Phase 3 CANNOT start without the keyword matrix from this phase.

1. Load `keyword_wisdom`. Compute confidence: `avg_score × log(times_used + 1)`.
2. Rank: Proven (conf ≥ 50, used ≥ 3) → Promising (30-49 or used < 3) → Underperformers (conf < 30, used ≥ 3).
3. Discovery budget: 1-2 NEW keywords from resume, hot patterns, interview JD language, adjacent titles.
4. Sample-before-commit (FULL_ANALYSIS only): 3 keywords at 2 companies, score 6 results. Swap if avg < 50.

**Display the matrix:**
```
KEYWORD MATRIX:
  Proven: "{kw}" (conf {X}), ...
  Promising: "{kw}" (conf {X}), ...
  Discovery: "{kw}" (new), ...
  Demoted: "{kw}" (conf {X}, skipping)
```

After run: update keyword_wisdom (times_used, avg_score, jobs_found, high_score_count, last_used, confidence).

---

## PHASE 2C: Market Health Check ← MANDATORY OUTPUT

Assess from recent data:
1. URL liveness rate (% 404 from last run)
2. New roles per company (3-run average)
3. Role freshness (avg days-since-posted)

Update `market_health` in preferences.

**If tightening** (url_404_rate > 0.4 OR avg_new_roles < 1 for 3+ runs):
- Propose strategy shift toward networking
- Increase Tier 3 discovery by 2
- Lower NETWORK FIRST threshold: Fit ≥ 65 (from ≥ 70)
- Add MARKET HEALTH to Daily Briefing

```
MARKET HEALTH:
  URL liveness: {X}% | Trend: {direction}
  New roles/company: {X} avg | Status: {status}
  {Actions taken if tightening}
```

---

## PHASE 2D: Company Intelligence

For Tier 1 + Tier 2 not checked in 7+ days, **actually run web searches** for each company (don't rely on existing intelligence from preferences alone — the point is to catch NEW developments):
- Web search for layoffs/freeze/growth news
- Set status: healthy / layoff_recent / hiring_freeze / high_growth
- Set hiring_momentum_modifier: -15 to +10
- Update preferences

FULL_ANALYSIS: Deep scan all targets. FAST_TRACK: Tier 1 only. QUICK_TRIAGE: Skip.

---

## PHASE 3: Multi-Channel Search

Use **keyword matrix from Phase 2B**. Read `references/career-sites.md` for tips.

### Browser Detection (do this FIRST)

Check if Chrome browser tools are available (`navigate`, `computer`, `read_page`, `javascript_tool`, `find`, `tabs_context_mcp`, `tabs_create_mcp`). If they are, read `references/linkedin-browser.md` now — it contains the complete browser-first workflow for Channels 2 and 4, which yields far more results than web search alone. Web search can only see what LinkedIn exposes to search engine crawlers (a small fraction of listings and almost none of the recruiter/HM posts). Browser mode unlocks the full LinkedIn experience.

**Channel 1 — Career Sites**: Navigate, run ALL matrix queries, paginate, extract, dedup.

**Channel 2 — LinkedIn Jobs**:
- **If browser tools available**: Follow `references/linkedin-browser.md` Channel 2 workflow — navigate LinkedIn Jobs directly, extract job cards via JavaScript, read promising JDs. Respect rate limits (3-5 sec waits, max 5 queries/session, stop on CAPTCHA).
- **If no browser tools**: Fall back to web search `site:linkedin.com/jobs "{query}"` — limited results but better than nothing.
- FT: 3 queries/2 pages. FA: all/3 pages.

**Channel 3 — Boolean/ATS**: Google `site:` operators for Greenhouse/Lever/Ashby. FT: 2 searches/page 1. FA: all/2 pages.

**Channel 4 — LinkedIn Recruiter/HM Posts** (highest-value channel — warm leads with 3-5x response rates):
- **If browser tools available**: Follow `references/linkedin-browser.md` Channel 4 workflow — search LinkedIn content feed directly for hiring/recruiting posts. **IMPORTANT**: LinkedIn's content search page uses obfuscated CSS and SVG rendering — JavaScript CSS selectors will NOT work. Use `find` and `read_page` (accessibility tree) to extract post authors, titles, and content. Extract poster info, classify as HM/recruiter/team member, generate post-referencing outreach drafts, add contacts to CRM. Scroll to load more posts (infinite scroll, no pagination buttons).
- **If no browser tools**: Fall back to web search `site:linkedin.com "hiring" "{query}"` — very limited since most posts aren't indexed.
- Extract signals, convert to leads with +15 IP. FT: 3 terms/10 posts. **FA: all terms/20 posts.**

If LinkedIn blocked (CAPTCHA, login wall, rate limit): log the specific error, report what was found so far, proceed with available channels. Never retry aggressively — the user's LinkedIn account is at stake.

**Cross-channel dedup** (4 layers): URL → normalized key → fuzzy title+location → job ID.

### ═══ CHANNEL COMPLETION GATE (BLOCKING) ═══

Phase 4 CANNOT start until this gate passes. The reason this gate exists is that historically, channels got silently dropped (especially Channel 4) and the user only discovered it after the run was complete. This wastes the user's time and misses potential warm-channel leads that are the most valuable outcomes.

Before moving to Phase 4, print this channel log and verify every channel was attempted:

```
CHANNEL COMPLETION LOG:
  Ch1 (Career Sites):         [✓/✗] — {X} roles found | {notes}
  Ch2 (LinkedIn Jobs):        [✓/✗/⚠] — {X} roles found | {browser/web_search} | {notes}
  Ch3 (Boolean/ATS):          [✓/✗] — {X} roles found | {notes}
  Ch4 (Recruiter/HM Posts):   [✓/✗/⚠] — {X} roles found | {browser/web_search} | {notes}
  Browser tools available:    [YES/NO]
  All channels attempted:     [YES/NO]
```

Note: ⚠ means the channel was attempted via web search fallback (browser unavailable). This is acceptable but yields significantly fewer results — especially for Channel 4 where most recruiter posts aren't indexed by search engines.

**If any channel shows ✗**: Go back and run it before proceeding. A channel returning 0 results is fine (mark ✓ with "0 roles found") — the requirement is that the search was *attempted*, not that it produced results. The only acceptable exception is if a platform is completely blocked/down (e.g., LinkedIn returns a login wall), in which case log the specific error.

**This applies to ALL modes** including QUICK_TRIAGE and FAST_TRACK. The scope per channel varies by mode (QUICK_TRIAGE does lighter searches), but every channel must be attempted. The warm-first philosophy depends on Channel 4 (recruiter/HM posts) being run every time — it's the primary source of warm-channel leads.

---

## PHASE 4: Read and Evaluate JDs

**No job scored until full description read.** For each new role:
1. Read complete description. Extract salary, posted date, velocity signals.
2. Stale filter: >120 days auto-SKIP, 31-60 days HOLD warning.
3. Save JD snapshot to `{workspace}/jd/`
4. Check cooldowns. Skip irrelevant (internships, PhD, wrong location/function).

**After processing all roles, print the stale filter summary:**
```
STALE FILTER: {X} auto-skipped (120+ days), {X} warned (31-60 days)
```

---

## PHASE 4.5: Extract Structured Signals ← NEW in v13, BLOCKING

**STOP: Read `references/signal_extraction.md` NOW** — it contains the exact signal block template you need to copy and fill in. Read it BEFORE producing any signal blocks, not during the same turn as scoring.

For each scoreable role, extract structured signals:
1. **Role signals**: seniority, leadership scope, IC vs management, team responsibility
2. **Domain signals**: primary and secondary domains, opportunity cluster tag
3. **Technology signals**: direct matches, transferable matches, gaps, learnable gaps
4. **Company signals**: growth stage, hiring expansion, team maturity, urgency

Produce a signal block for each role. Phase 5 CANNOT start without signal blocks.

---

## PHASE 5: Score Each Role

**Read `references/scoring.md` first.** Scores consume extracted signals from Phase 4.5 — not raw JD text.

Flow: Signal block → Fit Score → Opportunity Score → Interview Probability → Uplevel/Downlevel.

---

## PHASE 5.5: Evaluate Networking Path Strength + Update CRM

**Read `references/networking_strategy.md` AND `references/networking_crm.md` first.**

For each scored role:
1. Classify networking path: Strong / Moderate / Weak
2. Apply networking boost to Interview Probability (+20 Strong, +10 Moderate, +0 Weak)
3. Generate referral search plan for NETWORK FIRST and HOLD roles
4. **CRM Integration (NEW in v14)**:
   - For each contact identified, create or update a CRM contact record
   - Set networking pipeline state (DISCOVERED → CONTACT_IDENTIFIED if contact found)
   - Link contact to opportunity in `opportunity_pipelines`
   - Check existing CRM contacts first — reuse if already tracked (dedup by name+company)

---

## PHASE 6: Determine Engagement Strategy + Generate Action Pack

**Read `references/decision_engine.md` for reasoning requirements.**
**Read `references/guardrails.md` for hard constraints.**

Flow: Scores + networking path → Action Gate → Guardrails check → Reasoning block → Bucket assignment → Adaptive thresholds → Company expansion if needed.

**Each recommendation MUST include a reasoning block** explaining key signals, attractiveness, and strategy rationale.

Action Pack content:
- 6A-C: APPLY NOW — tailored bullets, keywords, headline, positioning, outreach, Reasons-to-Believe
- 6D: NETWORK FIRST — referral search plan, coffee chat draft, networking timeline, tailored bullets
- 6E: HOLD — reason, next action, referral suggestion

Save to `{workspace}/Job_Search_Action_Pack_{M-D-YYYY}.md`

---

## PHASE 7: Generate CSV + URL Verification

26-column schema (A-Z). See `references/output-formats.md`. Run URL verification — replace generic career page URLs with specific posting links.

**IMPORTANT**: The CSV columns MUST match the exact 26-column schema in `references/output-formats.md` § Column-to-Array Position Mapping. Each row must have exactly 26 values in the order: Search Date, Rank, Action, Job Title, Salary Range, **Location**, Company, Team, Posted Date, Fit Score, Opportunity Score, Interview Prob, Uplevel Risk, Downlevel Risk, Bucket, Key Strengths, Potential Gaps, URL, Stage, Next Action, Next Action Date, Applied, Interview, Referral, Resume Ver, JD Hash.

---

## PHASE 8: Summary Report ← USE EXACT TEMPLATE

Read Summary Report template in `references/output-formats.md`. ALL sections required: action breakdown, top roles, bucket mix, keyword performance, market health, queue breakdown, learning applied, channel performance, cluster insights, trend, files.

**NEW in v14 — Additional required sections:**
- **Relationship Table**: Display all active CRM contacts with status, strength, follow-up dates (see `references/networking_crm.md` § Relationship Table Output)
- **Interview Pipeline Dashboard**: Display all active interviews with stage, prep status, next steps (see `references/interview_pipeline.md` § Interview Pipeline Dashboard)
- **Networking Pipeline Summary**: Count of opportunities in each networking state (DISCOVERED through CLOSED)

---

## PHASE 9: Append to Google Sheet ← STRICT COLUMN MAPPING

**STOP: Read `references/output-formats.md` § Column-to-Array Position Mapping NOW.** This defines the exact 26-value array format. The reason this explicit mapping exists is that Run 16's Apps Script had a missing Location column and extra text fields, which shifted all data and corrupted 4 rows. The mapping table and validation code in the template prevent this.

Apps Script via Monaco API (see `references/output-formats.md`). Verify. Retry once, then CSV fallback.

### Pre-Write Column Validation (MANDATORY — NEW)

Before injecting the Apps Script, verify the data array for EACH row:
1. **Count**: Exactly 26 values per row (not 25, not 27)
2. **Position check**: Index [5] is Location (not Company), Index [6] is Company (not Team), Index [17] is URL (not a text description)
3. **Type check**: Indices [9], [10], [11] are integers (Fit, Opp, IP scores), not strings
4. **No extra fields**: Only 2 text description columns — [15] (Key Strengths) and [16] (Potential Gaps). All match/networking/strategy text goes into these two fields or into [19] (Next Action). Never add extra text columns.

### Post-Write Verification (MANDATORY)

The reason this step exists is that Run 9's Apps Script save failed silently, and without verification the data loss wasn't caught. Always confirm:

1. Check execution log for "Added {X} rows starting at row {Y}"
2. **Switch to the Google Sheet tab** and scroll to the last rows
3. Screenshot to confirm data appears correctly
4. **Column alignment check**: Verify that the Company name appears in column G (not F or H), and the URL appears in column R (not Q or S). This catches column-shift errors.
5. Spot-check: verify company name, Fit score, and Action of the last row match CSV
6. For FULL_ANALYSIS: Apply conditional formatting (see `references/output-formats.md` § Conditional Formatting)

---

## PHASE 10: Save Preferences + Update Learning

**STOP. Read `references/learning_loop.md` now.** It contains details on what each field means and how it feeds into the next run's scoring. Phase 10 is where the learning loop closes — skipping fields means the next run starts dumber than it should be.

### Phase 10 Field Checklist

Print this checklist and check off each field as you update it. The reason this explicit checklist exists is that Run 13's evaluation found 5 preference fields were silently skipped — `channel_performance`, `cluster_outcomes`, `weekly_kpis`, `referral_tracking`, and `sourcing_performance` — partially breaking the learning loop for subsequent runs.

```
PHASE 10 UPDATE CHECKLIST:
[ ] last_run → "{date} (run {N})"
[ ] run_counter → {N}
[ ] company_search_log → Update times_searched + new_roles_found_last for EACH company searched
[ ] company_intelligence → Update EXISTING keys only (NEVER create "Company_intel" duplicates)
[ ] keyword_wisdom → Update times_used, avg_score, confidence for each keyword used this run
[ ] market_health → url_404_rate_by_run, trend, dead_channels_this_run, recommendation
[ ] learning_history.strategy_changes → Append run summary
[ ] learning_history.weekly_kpis → Add new entry if 7+ days since last snapshot
[ ] referral_tracking → Add entries for NEW networking targets (Moderate/Strong paths found this run)
[ ] sourcing_performance → Update if new applications submitted; note "no change" if none
[ ] channel_performance → Update if new Applied/Interview outcomes; note "no change" if none
[ ] cluster_outcomes → Update if new applications submitted; note "no change" if none
[ ] last_sheet_write_status → "success_run{N}_{X}rows_at_{Y}" or "failed_{reason}"
--- CRM + INTERVIEW PIPELINE (NEW in v14) ---
[ ] networking_crm.contacts → Add new contacts discovered this run
[ ] networking_crm.contacts → Update outreach_status for contacts acted on
[ ] networking_crm.contacts → Expire 30+ day unresponsive contacts (downgrade strength)
[ ] opportunity_pipelines → Update networking_state for each active opportunity
[ ] opportunity_pipelines → Add state_history entries for state changes this run
[ ] interview_pipeline → Update stage for any interviews with new outcomes
[ ] interview_pipeline → Add new entries for roles that moved to interview
[ ] interview_pipeline → Close entries for completed processes
[ ] interview_pipeline → Generate prep tasks for newly scheduled interviews
[ ] interview_pipeline.thank_you_notes → Flag unsent thank-you notes
All fields checked: [ ] YES
```

**Clarification on "no change" fields**: Not every field changes every run. If no new applications were submitted this run, `sourcing_performance`, `channel_performance`, and `cluster_outcomes` won't change — but you still need to acknowledge them ("no change — 0 new apps this run") rather than silently skipping. This prevents the ambiguity of "was this field skipped or was it unchanged?"

---

## Operational Modes (NEW in v14)

The system now operates in two primary modes that determine which phases run:

### Discovery Mode
Runs the **full pipeline** including new role search (Phases 0 through 10). Use when the user wants to find new opportunities.

### Networking Mode
Runs **Phases 0, 1, 1.5, 1.7, 5.5 (CRM updates only), 8, and 10**. Skips new role discovery (Phases 2-5). Focuses on:
- Managing CRM follow-ups and relationship progression
- Updating interview pipeline and generating prep tasks
- Processing sheet changes (new applications, interviews, stage updates)
- Producing Relationship Table and Interview Pipeline Dashboard

### Mode Detection

| User says | Mode | Depth |
|-----------|------|-------|
| "run my job search" / "find new roles" / "deep dive" | Discovery | FULL_ANALYSIS |
| "daily search" / "check {company}" | Discovery | FAST_TRACK |
| "quick scan" / "anything new?" | Discovery | QUICK_TRIAGE |
| "networking update" / "check my follow-ups" / "any follow-ups due" | Networking | — |
| "run networking mode" / "manage my contacts" | Networking | — |
| "interview prep" / "check my pipeline" | Networking | — |
| "daily check-in" (with many active pipelines, few new roles) | **Auto-select Networking** | — |

### Dynamic Mode Selection

When the user says something ambiguous like "daily check-in" or "what should I do today?", the system should auto-select the mode:
- If ≥3 active networking contacts with pending follow-ups → default to **Networking Mode**
- If ≥2 active interviews → default to **Networking Mode** (focus on prep)
- If market_health shows tightening and <3 new roles last run → suggest **Networking Mode**
- Otherwise → default to **Discovery Mode** (FAST_TRACK)

Always tell the user which mode was selected and why. They can override.

## Discovery Depth Levels

| Feature | QUICK_TRIAGE | FAST_TRACK | FULL_ANALYSIS |
|---------|-------------|------------|---------------|
| Companies | Max 3 | Max 5 | Unlimited |
| Roles/company | 10 (page 1) | 15 (2-3 pages) | All pages |
| Scoring | IP only | Fit + Opp(3-pillar) + IP | Fit + Opp(5-pillar) + IP |
| Channels | Ch1-2 | **All 4 (Ch4 lighter)** | **All 4 (Ch4 full depth)** |
| Signal extraction | Skip | Light (domain + role only) | Full (all 4 categories) |
| Networking path eval | Skip | Strong/Weak only | Full (Strong/Moderate/Weak) |
| Reasoning blocks | Skip | Top 3 only | All scored roles |
| Guardrails | Basic | Standard | Full enforcement |
| Tailoring | None | Top 3 full, rest light | All APPLY NOW full |
| Sheet append | No | Yes | Yes + formatting |
| Discovery | None | 1-2 companies | 3-5 companies |
| Keyword Intel | Skip | Load & use | Full + sample |
| Market Health | Skip | Load & display | Full + responsive |
| Company Intel | Skip | Tier 1 only | All companies |
| CRM follow-ups | Yes (always) | Yes (always) | Yes (always) |
| Interview pipeline | Yes (always) | Yes (always) | Yes (always) |

**CRM and Interview Pipeline phases run in ALL modes and BOTH operational modes.** They are never skipped because networking follow-ups and interview prep are time-sensitive.

---

## Execution Pipeline Summary (v14)

```
PHASE 0:    Load preferences + CRM memory
PHASE 1:    Daily Check-in (learning engine + channel performance)
PHASE 1.5:  Manage networking follow-ups (NEW — CRM layer)
PHASE 1.7:  Update interview pipeline (NEW — prep tasks, reminders)
--- DISCOVERY MODE ONLY (skip in Networking Mode) ---
PHASE 2A:   Build search queue
PHASE 2B:   Keyword intelligence (BLOCKING GATE)
PHASE 2C:   Market health check
PHASE 2D:   Company intelligence
PHASE 3:    Multi-channel search (4 channels)
PHASE 4:    Read and evaluate JDs
PHASE 4.5:  Extract structured signals (BLOCKING GATE)
PHASE 5:    Score using signals (not raw JD text)
--- END DISCOVERY ONLY ---
PHASE 5.5:  Evaluate networking path strength + update CRM
PHASE 6:    Determine engagement strategy + Action Pack (with reasoning + guardrails)
PHASE 7:    Generate CSV
PHASE 8:    Summary Report + Relationship Table + Interview Pipeline Dashboard
PHASE 9:    Append to Google Sheet
PHASE 10:   Save preferences + update learning + update CRM + update interview pipeline
```

### Networking Mode Pipeline (condensed)

```
PHASE 0:    Load preferences + CRM memory
PHASE 1:    Daily Check-in (sync sheet changes, compute learning)
PHASE 1.5:  Manage networking follow-ups (primary focus)
PHASE 1.7:  Update interview pipeline (primary focus)
PHASE 5.5:  Update CRM for existing opportunities (no new scoring)
PHASE 8:    Relationship Table + Interview Pipeline Dashboard
PHASE 10:   Save all state (CRM, interview pipeline, learning)
```

---

## Deduplication (4 layers)

1. Exact URL → auto-skip
2. Normalized key (company|title|location) → auto-skip
3. Fuzzy title+location (>80% overlap) → HOLD "confirm duplicate"
4. Job ID extraction → auto-skip

Also check against Google Sheet rows.

---

## Stale Posting Thresholds

| Age | Action |
|-----|--------|
| 0-14 days | Full priority |
| 15-30 days | Normal |
| 31-60 days | HOLD warning, -10 IP |
| 61-120 days | -20 IP |
| 120+ days | Auto-SKIP (zombie) |

---

## Date Format

All dates: `M/D/YYYY` — no leading zeros.
