---
name: job-search
description: |
  **Personal CRM + Opportunity Intelligence System (v16.11)**: Three-domain platform — Opportunity Discovery (4-channel search, multi-ATS API network, ATS probe protocol, URL pre-triage, URL liveness, Fit/Opportunity/IP scoring, APPLY NOW/NETWORK FIRST/HOLD/BORDERLINE/SKIP gates), Relationship CRM (8-state networking pipeline DISCOVERED→CLOSED, follow-ups), Interview Pipeline (stage tracking, prep). Discovery and Networking modes. ATS registry grows automatically. Includes false-negative risk assessment, Borderline Review, company title mapping, Google application budget guardrail, Ch3 ATS priority ordering, context efficiency, Google URL freshness heuristic.

  Trigger when user mentions "jobs", "careers", "openings", "roles", "positions", "hiring", "apply", "find roles matching my resume", "daily job search", "run my job search", "find referrals", "who should I network with", "follow up", "check my pipeline", "interview prep", "networking update", "any follow-ups due", or "run networking mode".
---

# Personal CRM + Opportunity Intelligence System v16.11 — Execution Guide

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
- `references/output-formats.md` — CSV 27-column schema, Action Pack format, Summary Report template, Google Sheet write process, tailoring tiers, Relationship Table, Interview Pipeline Dashboard
- `references/career-sites.md` — Company-specific navigation tips, ATS patterns, LinkedIn/Boolean search templates
- `references/linkedin-browser.md` — **NEW in v15** — Chrome browser automation for LinkedIn Jobs (Channel 2) and Recruiter/HM Posts (Channel 4). Read before Phase 3 when browser tools are available.
- `references/linkedin-company-filters.md` — **NEW in v16.6** — LinkedIn `f_C` company ID registry for targeted job board searches. Maps companies to their verified LinkedIn IDs. Read before Phase 3 Channel 2 company-filtered searches.
- `references/keyword_engine.md` — **NEW in v16.11** — Full Phase 2B keyword matrix construction, company-specific title vocabulary (v16.7), global title expansion map (v16.11), two-query-type system. Read before Phase 2B.
- `references/channel_search.md` — **NEW in v16.11** — Complete Phase 3 multi-channel search execution (Ch1-4), Chrome MCP setup, ATS API details, seed registry, probe protocol, adaptive expansion, Channel Completion Gate. Read before Phase 3.
- `references/url_triage.md` — **NEW in v16.11** — Phases 3.5-3.7: URL pre-triage, early liveness sampling, cross-run dedup gate, Google URL freshness heuristic. Read before Phase 3.5.

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

[ ] PHASE 2B: Keyword Intelligence Engine (Enhanced in v16.11)
    → OUTPUT: Keyword matrix printed (proven + discovery keywords with confidence scores)
    → OUTPUT: Company keyword overrides printed (from title_vocabulary intelligence)
    → OUTPUT: Title expansion map loaded — adjacent titles added to Discovery tier (v16.11)
    → PERSIST: Save keyword matrix to {workspace}/keyword_matrix_run{N}.md (v16.11 — survives context continuations)
    → BLOCKING: Phase 3 cannot start without the keyword matrix

[ ] PHASE 2C: Market Health Check
    → OUTPUT: Market health assessment printed (url_404_rate, trend, responsive actions)
    → If tightening detected: lower NETWORK FIRST threshold (Fit ≥ 65), increase Tier 3 by 2, propose strategy shift

[ ] PHASE 2D: Company Intelligence (Enhanced in v16.7)
    → Step 1: Intelligence staleness check (0-3d fresh / 4-7d spot-check / 8+d full refresh)
    → Step 2: Career site spot-check for aging/stale companies
    → Step 3: Referral company override (always full refresh)
    → Step 4: Team-level freeze resolution for partial_freeze companies (v16.7)
    → OUTPUT: Company health status for each company in queue + staleness actions taken
    → OUTPUT: Team-level status for any partial_freeze companies

[ ] PHASE 2E: Context Budget Planning (NEW in v16.11) ← BLOCKING GATE
    → Step 1: Estimate context cost — count companies in search queue × avg JD tokens per company
      - Rule of thumb: each company = ~2000-4000 tokens (career site nav + JD reads)
      - Each ATS API query = ~500-1000 tokens; each full JD read = ~1500-3000 tokens
      - FULL_ANALYSIS with 20+ companies typically needs 80K+ tokens
    → Step 2: If estimated cost > 60% of remaining context → activate EFFICIENCY MODE:
      - Ch3: Query all 4 ATS platforms for title matches FIRST (lightweight), collect URLs only
      - Ch3: Deep JD reads only for title-matching roles (skip full JD for non-matches)
      - Ch1: Shallow reads for companies with >2 roles (read first role fully, skim rest for differentiation)
      - HOLD candidates: Shallow JD read (title + requirements + seniority only, skip company description + benefits)
      - Phase 4.0 Metadata Pre-Filter: Apply AGGRESSIVELY — skip anything with >1 concern signal
    → Step 3: If estimated cost < 60% → proceed normally with full reads
    → OUTPUT: Context budget assessment — {NORMAL / EFFICIENCY MODE} — estimated {X}% context usage
    → BLOCKING: Phase 3 CANNOT start without this assessment printed. Run 16 hit context limits because this phase was skipped.
    → ENFORCEMENT: If EFFICIENCY MODE activated, print "⚠ EFFICIENCY MODE ACTIVE" at the top of Phase 3, 4, and 4.5 as a reminder to use shallow reads

[ ] PHASE 3: Multi-Channel Search (Ch1-3 required; Ch4 requires browser)
    → Channel 1 (Career Sites): [ ] completed — {X} roles found
    → Ch1 Adaptive Expansion (v16.7): [ ] {X} companies triggered fallback keywords — {Y} new roles found
    → Channel 2 (LinkedIn Jobs): [ ] completed — {X} roles found (title: {X}, tool: {X}, freshness: {X})
    → Ch3 PRIORITY ORDER (v16.11): Query Lever + SmartRecruiters FIRST (most likely to be skipped in context-constrained runs), THEN Ashby, THEN Greenhouse (largest registry, can be truncated if context runs low)
    → Channel 3 (ATS Boards): [ ] completed — {X} roles found (Greenhouse: {X}, Ashby: {X}, Lever: {X}, SmartRecr: {X}, site-search: {X})
    → ATS Probes: [ ] {X} new companies probed — {X} added to registry
    → Channel 4 (Recruiter/HM Posts): [ ] completed/skipped — {X} roles found | browser: {YES/NO}
    → Cross-channel dedup: [ ] completed — {X} unique roles after dedup
    → CHANNEL COMPLETION GATE: [ ] All active channels attempted (print log before Phase 3.5)

[ ] PHASE 3.5: URL Pre-Triage (NEW in v16)
    → LinkedIn ID age filter applied — {X} dropped
    → Career site URL swaps — {X} swapped
    → Pattern-based rejection — {X} dropped
    → OUTPUT: {X} roles surviving → Phase 3.6

[ ] PHASE 3.6: Early URL Liveness Sampling (NEW in v16.6)
    → TRIGGER: Run if prev url_404_rate > 0.3 OR any company has freeze/migration flags
    → Step 1: Group URLs by company, sample 1 per company
    → Step 2: Quick liveness check (navigate + 3s wait + screenshot)
    → Step 3: Flag EARLY_DEAD companies (deprioritize their JD reads)
    → Step 4: Detect PLATFORM_MIGRATION signals
    → Step 5 (NEW v16.11): Google URL Freshness Pre-Check — For ALL Google URLs found via web search (not direct career site browsing), immediately liveness-check before scoring. Google career URLs found via web search have a 75% dead rate (Run 15 data: 3 of 4 dead). If dead, DROP immediately — do not score or include in downstream phases. This saves significant context on roles that don't exist.
    → TRIGGER: Always run for Google. Consider for any company with url_404_rate > 0.5 in last 3 runs.
    → OUTPUT: {X} companies healthy, {Y} flagged EARLY_DEAD, {Z} migration detected

[ ] PHASE 3.7: Cross-Run Dedup Gate (NEW in v16.5 — moved from Phase 7.5)
    → Step 1: Read existing Google Sheet rows → extract Company+Title+URL for dedup
    → Step 2: Check each role against sheet → drop exact URL matches, Company+Title matches, fuzzy matches
    → OUTPUT: Dedup log printed — {N} dropped, {M} passed
    → BLOCKING: Only roles passing dedup proceed to Phase 4 (JD reading)
    → WHY HERE: Deduping before JD reading saves 40-60% of context on roles already in sheet

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
    → OUTPUT: Borderline Review applied for Bucket A roles with Fit < 70 AND Opp ≥ 75 (v16.11)
    → OUTPUT: Adaptive thresholds applied if needed
    → OUTPUT: If <5 APPLY NOW at floor → company expansion triggered: [ ] yes/no
    → NEW Guardrail 8 (v16.11): Google Application Budget — check `company_intelligence.Google.application_budget.slots_available` before assigning APPLY NOW to any Google role. If 0 slots → force HOLD. If 1 slot → only the highest-Fit Google role can receive APPLY NOW.

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
    → **6F (NEW v16.11): Pre-compute csv_row arrays** — build 27-value array per scored role NOW while context is fresh
    → OUTPUT: csv_row array stored per role (used in Phase 7, 8.7, and 9)

[ ] PHASE 7: Generate CSV (uses pre-computed csv_rows from Phase 6F)
    → OUTPUT: CSV saved with EXACTLY 27 columns (A-AA) per output-formats.md § Column-to-Array Position Mapping
    → VERIFY: Each row has 27 values; Location is at index [5], Company at [6], URL at [17]
    → NOTE: Phase 7 now assembles the CSV from pre-built csv_row arrays — no re-reading of scoring data needed

[ ] PHASE 7.5: URL Liveness Gate (BLOCKING) — dedup moved to Phase 3.7 in v16.5
    → Step 1: For each role, navigate to URL and confirm live job posting loads
    → Step 2: Dead LinkedIn URLs → attempt career site replacement. No replacement → DROP role
    → OUTPUT: Liveness log printed — {N} live, {M} replaced, {K} dropped
    → REGENERATE: CSV and Action Pack with only surviving roles (renumber ranks)
    → BLOCKING: Phase 8 and 9 CANNOT start until this gate passes

[ ] PHASE 8: Summary Report (use EXACT template from references/output-formats.md)

[ ] PHASE 8.5: Coverage Report (NEW in v16.2 — MANDATORY)
    → OUTPUT: Search Reach printed (companies searched, APIs queried, LinkedIn mode)
    → OUTPUT: Channel Health status for all 4 channels
    → OUTPUT: Blind Spots list (what this run could NOT see)
    → OUTPUT: False Negative Risk level assigned (LOW / MEDIUM / HIGH)
    → OUTPUT: Confidence Assessment with specific recommendation if risk > LOW
    → OUTPUT: Summary displayed with ALL sections
    → OUTPUT: Relationship Table displayed (contacts, statuses, follow-ups)
    → OUTPUT: Interview Pipeline Dashboard displayed

[ ] PHASE 8.7: Write Checkpoint JSON (NEW v16.11 — MANDATORY before sheet write)
    → OUTPUT: scored_run{N}.json saved to {workspace}/
    → Contains: all scored roles with pre-computed csv_row arrays, skipped roles, action_summary, url_liveness_log, coverage_report
    → WHY: If Phase 9 fails, this checkpoint lets you retry the sheet write without re-doing Phases 1-8
    → FALLBACK: If Phase 9 fails after retry, tell user "run scored_run{N}.json available — say 'sync run {N}' to retry the sheet write via job-sync skill"

[ ] PHASE 9: Append to Google Sheet (read output-formats.md § Column-to-Array Mapping FIRST)
    → SOURCE: Read csv_row arrays from scored_run{N}.json (Phase 8.7) — do NOT reconstruct from memory
    → v16.5: Use pre-built template from scripts/sheet_writer_template.js (see scripts/inject_and_run.py)
    → PRE-WRITE: Validate each data row has exactly 27 values in correct column order
    → PRE-WRITE: Confirm Location at [5], Company at [6], scores as integers at [9-11], URL at [17]
    → OUTPUT: {X} rows written, verified
    → POST-WRITE: Switch to sheet tab, screenshot last rows to confirm data + column alignment
    → FULL_ANALYSIS: Apply conditional formatting (see output-formats.md)
    → ON FAILURE: Retry once. If still fails → direct user to job-sync fallback (see Phase 8.7)

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

Always save after successful run. Key fields: `last_run`, `run_counter`, `learning_history`, `keyword_wisdom`, `market_health`, `company_search_log`, `application_cooldowns`, `sourcing_performance`, `company_intelligence`, `referral_tracking`, `channel_performance`, `cluster_outcomes`, `networking_crm`, `opportunity_pipelines`, `interview_pipeline`, `ats_registry`.

The preferences schema includes:
- **Core**: `candidate_profile`, `location_preferences`, `target_companies`, `google_sheet_url`, `minimum_salary_base`, `search_channels`
- **Intelligence**: `company_intelligence`, `keyword_wisdom`, `application_cooldowns`, `market_health`
- **Learning**: `learning_history` (with conversion_rates, hot_patterns, cold_patterns, black_hole_companies, score_adjustments, weekly_kpis, strategy_changes, channel_performance, cluster_outcomes)
- **CRM (NEW in v14)**: `networking_crm` (with contacts[]), `opportunity_pipelines` (with per-opportunity networking state tracking) — see `references/networking_crm.md`
- **Interview Pipeline (NEW in v14)**: `interview_pipeline[]` (with per-opportunity interview stage, prep tasks, thank-you tracking) — see `references/interview_pipeline.md`
- **ATS Registry (NEW in v16.2)**: `ats_registry` — Maps companies to their ATS platform, API slug, and query history. Grows automatically via ATS Probe protocol. See Channel 3 for details.
- **Title Vocabulary (NEW in v16.7)**: `company_intelligence.{company}.title_vocabulary` — Per-company keyword mapping learned from career site search results. Contains `high_yield` (keywords that return many results), `low_yield` (keywords that return near-zero), `learned_from` (run that discovered it), `last_updated`. Built automatically via adaptive keyword expansion in Ch1.
- **Team-Level Status (NEW in v16.7)**: `company_intelligence.{company}.team_status` — Granular hiring status per team/org within a company. Replaces blanket freeze modifiers with targeted team-specific modifiers when a company is `partial_freeze`.
- **Title Expansion Map (NEW in v16.11)**: `title_expansion_map` — Global adjacent-title families that the pipeline searches alongside primary titles. Catches market title fragmentation (e.g., "Analytics Engineering Manager" as adjacent to "Data Science Manager"). Grows automatically when runs discover high-Fit roles under new title variants.
- **Borderline Review (NEW in v16.11, UPDATED v16.11)**: Bucket A roles with Fit < 70 and Opp ≥ 75 receive HOLD — BORDERLINE instead of plain HOLD or auto-SKIP. Expanded from Fit 58-64 to catch high-value near-misses just below NETWORK FIRST threshold (e.g., Pinterest Director Fit 68/Opp 85). See scoring.md § Borderline Review.
- **Google Application Budget (NEW in v16.11)**: `company_intelligence.Google.application_budget` — Tracks Google's rolling application limit. Fields: `max_active_apps` (default 4), `slots_used`, `cooldown_after_rejection_days` (default 90), `active_applications[]` (with role, date, status), `last_rejection_date`, `slots_available`. Guardrail 8 enforces: if `slots_available == 0`, force HOLD with note "Google application budget exhausted — wait until {date}". Prefer NETWORK FIRST for Google roles to avoid wasting limited application slots.
- **Networking Execution Tracking (NEW in v16.11)**: New column V ("Networked") in Google Sheet tracks whether networking outreach was actually executed for a role. This closes the feedback loop between NETWORK FIRST recommendations and actual actions taken. Phase 1 reads this column to compute networking execution rate. Schema is now 27 columns (A-AA).
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
6. New Networked = TRUE rows (v16.11) — track networking execution rate

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
- **Networking execution rate (v16.11)**: Count NETWORK FIRST roles with Networked=TRUE vs total NETWORK FIRST. Target: >80%. If <50%, surface in Daily Briefing as concern.
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

**STOP: Read `references/keyword_engine.md` NOW.** It contains the full keyword matrix construction process, company-specific title vocabulary (v16.7), global title expansion map (v16.11), and two-query-type system (title-based + tool-based).

Phase 3 CANNOT start without the keyword matrix from this phase.

Quick summary:
1. Load `keyword_wisdom`, compute confidence, rank: Proven → Promising → Underperformers
2. Load company keyword overrides from `title_vocabulary`
3. Load `title_expansion_map` for adjacent titles
4. Build per-company keyword plan
5. Discovery budget: 1-2 new keywords

→ OUTPUT: Keyword matrix, company overrides, title expansion map
→ PERSIST (NEW in v16.11): Save the keyword matrix output to `{workspace}/keyword_matrix_run{N}.md`. This ensures the matrix survives context continuations. Run 16 lost this artifact when context ran out between Phase 2B and Phase 10. The file should contain: (1) full keyword matrix with confidence scores and tiers, (2) company keyword overrides, (3) title expansion map, (4) discovery keywords attempted this run

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

## PHASE 2D: Company Intelligence ← Enhanced in v16.7

### Step 1: Intelligence Staleness Check (MANDATORY — run BEFORE web searches)

**Why this exists**: Run 11 had DoorDash marked "healthy" (checked 3/8) when their entire DS Analytics team had frozen hiring, and LinkedIn marked "layoff_recent" with -15 modifier when they had 312 active openings. Stale intelligence fed wrong modifiers into scoring, causing the pipeline to miss real opportunities and waste time on dead ones. This step forces a reality check.

**For EVERY company in the search queue (not just 7+ days stale):**

1. **Check `company_intelligence.{company}.last_checked`** — compute days since last check
2. **Apply staleness tiers:**
   - **0-3 days**: Fresh — trust existing intelligence, skip web search
   - **4-7 days**: Aging — quick career site spot-check (navigate to career site, search your top keyword, check if results load and count roughly matches expectations). If results are dramatically different from last check (e.g., 0 results when last check found 10+), escalate to full refresh
   - **8+ days**: Stale — MANDATORY full refresh (web search + career site spot-check)
   - **Never checked or missing**: Treat as stale

3. **Career site spot-check** (the key addition — takes 30 seconds per company):
   - Navigate to the company's career site
   - **v16.7**: Use the company's `title_vocabulary.high_yield[0]` if available, otherwise use your #1 default keyword (e.g., "data science"). This prevents false-freeze detection caused by keyword mismatch — e.g., DoorDash returns 0 for "data science" but 13 for "analytics", so using the wrong keyword would falsely trigger a freeze flag.
   - Count results. Compare to `company_search_log.{company}.new_roles_found_last`
   - **If results = 0 and previously found roles > 0** → likely hiring freeze or reorg. But FIRST try 1-2 alternative keywords before declaring a freeze (this is how vocabulary gets discovered). Mark `status: "partial_freeze"` or `"hiring_freeze"` only after alternatives also return 0, set `hiring_momentum_modifier` to -10 or -15
   - **If results >> previous** → company is ramping up. Consider upgrading status to `"high_growth"`
   - **If career site errors/redirects/restructured** → flag for platform migration check (see Phase 10)

4. **Priority company override**: Companies where the candidate has a referral (`referral_tracking`) or active networking contacts ALWAYS get a full refresh regardless of age. The cost of stale intelligence is highest for warm-channel companies.

### Step 2: Web Search Intelligence (existing behavior, refined)

For companies requiring refresh (per Step 1), **actually run web searches** (don't rely on existing intelligence from preferences alone — the point is to catch NEW developments):
- Web search for layoffs/freeze/growth news
- Set status: healthy / layoff_recent / hiring_freeze / partial_freeze / high_growth
- Set hiring_momentum_modifier: -15 to +10
- Update `last_checked` to today's date
- Update preferences

**Status definitions** (expanded in v16.6):
- `healthy` — Normal hiring, no negative signals
- `high_growth` — Expanding aggressively, multiple new headcount signals
- `layoff_recent` — Layoffs in past 60 days, hiring may be impacted but not frozen
- `partial_freeze` — Some teams frozen, others still hiring (e.g., DoorDash DS Analytics freeze but Ads still active)
- `hiring_freeze` — Company-wide freeze confirmed, skip career site search

### Granular Team-Level Status ← NEW in v16.7

**Why this exists**: Run 12's DoorDash diagnostic found that a blanket `partial_freeze` with `-10` modifier penalized the entire company, when only the core DS Analytics team was frozen — the Ads & Promotions Analytics team was actively hiring (role reposted 2 weeks ago, 100+ LinkedIn applicants). A company-wide modifier caused the pipeline to deprioritize a company that had real, active opportunities in a different team.

**When `partial_freeze` is detected, add team-level detail:**

```json
"company_intelligence": {
  "DoorDash": {
    "status": "partial_freeze",
    "team_status": {
      "DS Analytics (Core)": {"status": "freeze", "modifier": -15},
      "Ads & Promotions Analytics": {"status": "active_hiring", "modifier": 0},
      "Strategy & Operations": {"status": "active_hiring", "modifier": 0}
    },
    "hiring_momentum_modifier": -5
  }
}
```

**How team-level status affects scoring:**
- When a role matches a team in `team_status`, apply THAT team's modifier instead of the company-wide modifier
- When a role's team is unknown or not listed, apply the company-wide modifier as fallback
- The company-wide `hiring_momentum_modifier` should reflect the WEIGHTED picture (e.g., -5 instead of -10 when some teams are frozen but others are active)
- During Phase 4.5 signal extraction, attempt to identify which team/org the role belongs to (from JD's "About the Team" section) and match it against `team_status` keys

### Step 3: Print intelligence summary

```
COMPANY INTELLIGENCE (Phase 2D):
  Checked: {X} companies
  Fresh (0-3d): {list} — trusted existing intel
  Spot-checked (4-7d): {list} — {results}
  Full refresh (8+d): {list} — {results}
  Referral overrides: {list} — forced full refresh
  Status changes: {list of any changes from previous status}
```

FULL_ANALYSIS: Deep scan all targets. FAST_TRACK: Tier 1 + referral companies. QUICK_TRIAGE: Referral companies only.

---

## PHASE 3: Multi-Channel Search

**STOP: Read `references/channel_search.md` NOW.** It contains the complete 4-channel search execution guide including Chrome MCP setup, career site workflows, LinkedIn browser automation, ATS API details (Greenhouse/Ashby/Lever/SmartRecruiters), seed registry, ATS probe protocol, adaptive keyword expansion (v16.7), and Channel Completion Gate.

Also read: `references/career-sites.md` (navigation tips), `references/linkedin-browser.md` (Ch2/Ch4 browser workflows), `references/linkedin-company-filters.md` (company ID registry).

**Channel overview:**
- **Ch1 (Career Sites)** — MANDATORY. Chrome MCP is default. Apple always first. Proprietary sites.
- **Ch2 (LinkedIn Jobs)** — Chrome MCP preferred. Title + tool queries. Freshness sweep first.
- **Ch3 (ATS Boards)** — API calls to Greenhouse, Ashby, Lever, SmartRecruiters. **v16.11 PRIORITY: Lever + SmartRecruiters FIRST**, then Ashby, then Greenhouse.
- **Ch4 (Recruiter/HM Posts)** — Chrome REQUIRED. Feed → notifications → search → company pages.

**Channel Completion Gate (BLOCKING):** Print channel log before Phase 3.5.

---
## PHASE 3.5–3.7: URL Pre-Triage, Liveness Sampling, Cross-Run Dedup

**STOP: Read `references/url_triage.md` NOW.** It contains the complete URL triage pipeline:

- **Phase 3.5**: LinkedIn ID age filter, career site URL preference, pattern rejection, known-problematic flagging.
- **Phase 3.6**: Stratified 1-per-company sampling, EARLY_DEAD flagging, platform migration detection. **v16.11: Google URL Freshness Pre-Check** (75% dead rate).
- **Phase 3.7**: Cross-run dedup against Google Sheet. BLOCKING — only unique roles proceed to Phase 4.

These gates save 40-80% of downstream context.

---
## PHASE 4: Read and Evaluate JDs

### Phase 4.0: Metadata Pre-Filter (FAST SKIP) ← NEW in v16.11

**Before reading any full JDs**, do a quick pass over the metadata already available from Ch1-3 (title, company, location, department). Many roles can be eliminated from title and metadata alone — saving the expensive full JD read for roles that have a realistic chance of scoring above HOLD floor.

**Auto-SKIP without reading JD if ANY of these apply:**
- **Title signals IC role**: Contains "Lead" without "Manager/Director/Head", or "Senior [Individual Role]", "Staff", "Principal" — and candidate is leadership-track (Guardrail 7)
- **Title signals wrong function**: Contains "Engineer", "Developer", "Designer", "Recruiter", "People Analytics", "Marketing Analytics" (unless these are in candidate's target functions)
- **Title signals uplevel**: Contains "VP", "Chief", "President", "Head of" at companies where these are C-suite equivalent
- **Location mismatch**: Role is clearly non-US or in-office-only at a non-target city (from metadata)
- **Company blocked**: Company is in blocked list (e.g., Amazon)
- **Comp clearly below floor**: If salary visible in metadata and max < $180K

**Print the pre-filter summary:**
```
METADATA PRE-FILTER: {X} roles entered → {Y} fast-SKIPped → {Z} proceeding to full JD read
  Fast-SKIPped: {title} @ {company} — {reason}
  ...
```

This typically saves 30-50% of JD reads. In Run 14, 5 of 9 roles (Discord DE, Instacart People Analytics, Airbnb IC, Affirm IC, Lyft IC) could have been caught here.

### Phase 4.1: Full JD Read

**No job scored until full description read.** For each role surviving the pre-filter:
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

**v16.5 Cold APPLY NOW check**: Before assigning APPLY NOW to any role with no warm channel (no Strong networking path), verify IP ≥ 55. If IP < 55 AND no warm channel → downgrade to NETWORK FIRST with note "Cold channel + IP {X} below floor — find referral to improve odds." See `references/scoring.md` § Cold APPLY NOW IP Floor.

**Each recommendation MUST include a reasoning block** explaining key signals, attractiveness, and strategy rationale.

Action Pack content:
- 6A-C: APPLY NOW — tailored bullets, keywords, headline, positioning, outreach, Reasons-to-Believe
- 6D: NETWORK FIRST — referral search plan, coffee chat draft, networking timeline, tailored bullets
- 6E: HOLD — reason, next action, referral suggestion

Save to `{workspace}/Job_Search_Action_Pack_{M-D-YYYY}.md`

### Phase 6F: Pre-compute csv_row Arrays ← NEW in v16.11

**Why this matters**: The sheet write (Phase 9) is the most failure-prone step in the pipeline — it runs at the tail end of a long session when context is most strained. Previous runs have produced malformed rows, dropped columns, and column-shift errors because the 27-value arrays were being assembled from memory at the worst possible time.

The fix: build each role's csv_row immediately after scoring and action gate assignment, while all the signals, scores, and decisions are fresh in context. Phase 7 (CSV), Phase 8.7 (checkpoint), and Phase 9 (sheet write) then simply consume these pre-built arrays — no reconstruction needed.

**For each scored role** (not SKIPs), assemble a 27-value array following the exact column mapping in `references/output-formats.md` § Column-to-Array Position Mapping:

```
csv_row = [
  search_date,        # [0]  A: "M/D/YYYY"
  rank,               # [1]  B: integer
  action_gate,        # [2]  C: "NETWORK FIRST" etc
  job_title,          # [3]  D: full title
  salary_range,       # [4]  E: "200000-300000" or "N/A"
  location,           # [5]  F: location string
  company,            # [6]  G: company name
  team,               # [7]  H: team/org
  posted_date,        # [8]  I: "M/D/YYYY" or "N/A"
  fit_score,          # [9]  J: integer
  opp_score,          # [10] K: integer
  interview_prob,     # [11] L: integer
  uplevel_risk,       # [12] M: "LOW"/"MED"/"HIGH"
  downlevel_risk,     # [13] N: "LOW"/"MED"/"HIGH"
  bucket,             # [14] O: "A"/"B"/"C"
  key_strengths,      # [15] P: consolidated string
  potential_gaps,     # [16] Q: consolidated string
  url,                # [17] R: verified URL
  stage,              # [18] S: "New"
  next_action,        # [19] T: action string
  next_action_date,   # [20] U: "M/D/YYYY"
  false,              # [21] V: Networked (checkbox)
  false,              # [22] W: Applied (checkbox)
  false,              # [23] X: Interview (checkbox)
  false,              # [24] Y: Referral (checkbox)
  "",                 # [25] Z: Resume Ver
  jd_hash             # [26] AA: JD hash
]
```

**Validation**: After building each csv_row, immediately count elements — must be exactly 27. Verify [5] is Location, [6] is Company, [9-11] are integers. This catches errors while the data is still in front of you, rather than discovering them at Phase 9 when it's too late.

Store the csv_row alongside each role's scoring data. These arrays flow into Phase 7 (CSV file), Phase 8.7 (checkpoint JSON), and Phase 9 (sheet write) without modification.

---

## PHASE 7: Generate CSV + URL Verification

27-column schema (A-AA). See `references/output-formats.md`. Run URL verification — replace generic career page URLs with specific posting links.

**v16.11**: Phase 7 now assembles the CSV directly from the pre-computed csv_row arrays built in Phase 6F. Each role already has its validated 27-value array — Phase 7 simply writes them out as CSV rows. This eliminates the previous failure mode where column data was reconstructed from scattered context late in the session.

**IMPORTANT**: The CSV columns MUST match the exact 27-column schema in `references/output-formats.md` § Column-to-Array Position Mapping. Each row must have exactly 27 values in the order: Search Date, Rank, Action, Job Title, Salary Range, **Location**, Company, Team, Posted Date, Fit Score, Opportunity Score, Interview Prob, Uplevel Risk, Downlevel Risk, Bucket, Key Strengths, Potential Gaps, URL, Stage, Next Action, Next Action Date, Networked, Applied, Interview, Referral, Resume Ver, JD Hash.

---

## PHASE 7.5: URL Liveness Gate ← BLOCKING (v16.5: dedup moved to Phase 3.7)

This gate exists because of repeated, painful failures: Run 28 had 10 dead LinkedIn URLs that the user had to manually delete. Every role that makes it past this gate must have a verified-live URL. Roles that fail get dropped before the sheet write — it's far better to write 5 clean rows than 11 rows the user has to clean up.

**Note**: Cross-run deduplication was moved to Phase 3.7 in v16.5 to avoid wasting context on roles already in the sheet. By this point, all roles have already passed the dedup gate.

### Step 1: URL Liveness Verification (MANDATORY for ALL roles, not just APPLY NOW)

For every role in the pipeline, verify its URL actually loads a real job posting. Dead URLs waste the user's time — they click through and get a 404 or generic page. LinkedIn `/jobs/view/` URLs are the worst offenders (they die within days as listings close).

**Roles flagged as `URL_UNVERIFIED` in Phase 3.5** (known-problematic sources like Anthropic GH, CareerPuck, Chime GH) should skip full browser verification here — the issue is already documented. Just carry the flag through to output. Focus verification effort on URLs that haven't been pre-triaged.

**How to verify (for non-URL_UNVERIFIED roles):**

1. **If browser tools are available**: Navigate to each URL. Check for:
   - 404 / "page not found" / "this job is no longer available" / "no longer accepting applications"
   - Redirect to a generic search page or company homepage
   - LinkedIn login wall blocking the content

2. **If no browser tools**: Use `WebFetch` on the URL. Look for 404 status, redirect chains, or generic page content.

3. **For LinkedIn URLs** (most failure-prone):
   - Always try to find the equivalent company career site URL first — these are far more stable
   - If LinkedIn URL is dead: search `site:{company-careers-domain} "{exact job title}"` for replacement
   - If replacement found → use it. If not → **drop the role**

4. **For Greenhouse/Lever/Ashby URLs**: Generally reliable but still check for expired postings. If a new company's GH/Lever URLs systematically fail here, flag them for addition to the Phase 3.5 known-problematic list in Phase 10.

**URL Classification (assign one to each role):**
- `LIVE_JD` — URL loads and shows the specific job description → **PASS**
- `LIVE_REPLACED` — Original dead but replacement found on career site → **PASS** with updated URL
- `SEARCH_REDIRECT` — URL redirects to a search/browse page, not the specific JD → **DROP** (or keep as URL_UNVERIFIED if the role data came from API)
- `404_GONE` — URL returns 404 or "job no longer available" → **DROP**
- `403_BLOCKED` — URL returns 403 forbidden → **DROP** unless career site replacement found
- `JS_REQUIRED` — Page loads but content requires JavaScript rendering → keep as **URL_UNVERIFIED**
- `URL_UNVERIFIED` — Pre-flagged from Phase 3.5 known-problematic list → **PASS** (already flagged)

**Outcome for each URL:**
- `LIVE_JD` or `LIVE_REPLACED` → **PASS**
- `URL_UNVERIFIED` or `JS_REQUIRED` → **PASS with flag** (include in output, note in Potential Gaps)
- `SEARCH_REDIRECT`, `404_GONE`, `403_BLOCKED` → **DROP** (do not write to sheet) unless replacement found

**Display the liveness results:**
```
URL LIVENESS GATE:
  Tested: {X} URLs ({Y} pre-flagged URL_UNVERIFIED, {Z} newly verified)
  LIVE_JD: {N} ✓
  LIVE_REPLACED: {M} (found alternative URLs)
  URL_UNVERIFIED: {U} (known-problematic or JS-required, included with flag)
  Dropped: {K} ✗
    - {Company} — {Title}: {classification} (e.g., "404_GONE", "SEARCH_REDIRECT to careers page")
  Final output: {F} roles → proceeding to Phase 8
```

### Rate Limiting

When testing URLs in browser, wait 2-3 seconds between navigations. If LinkedIn returns CAPTCHA or rate limit, stop and mark remaining LinkedIn URLs as "unverified" with a warning.

### Regenerate outputs after this gate

After both checks complete, regenerate the CSV and Action Pack using ONLY the surviving roles. Renumber ranks based on surviving roles. The Summary Report (Phase 8) should reflect the post-gate counts, not the pre-gate counts. This ensures the sheet, CSV, and Action Pack all agree on the same set of roles.

### This gate is BLOCKING

Phase 8 and Phase 9 CANNOT proceed until both dedup and URL liveness are complete.

---

## PHASE 8: Summary Report ← USE EXACT TEMPLATE

Read Summary Report template in `references/output-formats.md`. ALL sections required: action breakdown, top roles, bucket mix, keyword performance, market health, queue breakdown, learning applied, channel performance, cluster insights, trend, files.

**NEW in v14 — Additional required sections:**
- **Relationship Table**: Display all active CRM contacts with status, strength, follow-up dates (see `references/networking_crm.md` § Relationship Table Output)
- **Interview Pipeline Dashboard**: Display all active interviews with stage, prep status, next steps (see `references/interview_pipeline.md` § Interview Pipeline Dashboard)
- **Networking Pipeline Summary**: Count of opportunities in each networking state (DISCOVERED through CLOSED)

---

## PHASE 8.5: Coverage Report ← NEW in v16.2 (MANDATORY)

**Why this exists**: The worst outcome for a daily job search system is a false negative — the tool reports "nothing out there" when opportunities exist but were missed due to tool limitations, blocked sites, or degraded channels. This report ensures the user can always distinguish between "quiet market" and "tool hit a wall." It must be printed at the end of every run, regardless of how many roles were found.

**Print the following report EXACTLY — every section is required:**

```
═══ COVERAGE REPORT — Run {N} ═══

SEARCH REACH:
  Companies directly searched (Ch1): {X} (list names)
  ATS API companies queried (Ch3):   {X} total
    Greenhouse: {X} queried of {Y} in registry
    Ashby:      {X} queried of {Y} in registry
    Lever:      {X} queried of {Y} in registry
    SmartRecr:  {X} queried of {Y} in registry
  ATS Registry size:                 {X} companies ({Y} with working APIs, {Z} unknown)
  New companies probed this run:     {X} ({results})
  LinkedIn queries executed:         {X} (browser: {YES/NO}, mode: {FULL/DEGRADED})
  ATS site: searches executed:       {X} (for companies not on any API)

CHANNEL HEALTH:
  Ch1 (Career Sites):    {✅ FULL / ⚠ PARTIAL / ❌ BLOCKED} — {X} roles — {notes}
  Ch2 (LinkedIn Jobs):   {✅ FULL (browser) / ⚠ DEGRADED (web search) / ❌ BLOCKED} — {X} roles
  Ch3 (ATS Boards):      {✅ FULL / ⚠ PARTIAL} — API: {X} roles, site-search: {X} roles
  Ch4 (Recruiter Posts): {✅ FULL (browser) / ⏭ SKIPPED (no browser)} — {X} roles

BLIND SPOTS (things this run could NOT see):
  {List each limitation honestly. Examples:}
  - LinkedIn Jobs searched via web fallback only — estimated 80-90% of listings invisible
  - Ch4 skipped — all recruiter/HM post leads missed this run
  - Netflix career site blocked (anti-bot) — 0 visibility into Netflix roles
  - {X} target companies not on any ATS API — only reachable via career site or LinkedIn
  - Greenhouse API companies not queried this run: {list}

FALSE NEGATIVE RISK: {LOW / MEDIUM / HIGH}
  - LOW: Browser available, all 4 channels active, 20+ ATS APIs queried across platforms
  - MEDIUM: No browser (Ch2 degraded, Ch4 skipped) but ATS API coverage strong (15+ companies)
  - HIGH: No browser AND fewer than 10 ATS APIs queried — significant opportunity gaps likely

CONFIDENCE ASSESSMENT:
  "This run searched {X}% of the addressable market for your profile.
   {If HIGH risk}: ⚠ There may be relevant roles this run could not reach."

SPECIFIC ACTIONS TO REDUCE RISK (list ALL that apply):
  - [ ] {action 1 — e.g., "Re-run with browser tools enabled for full LinkedIn access (Ch2+Ch4)"}
  - [ ] {action 2 — e.g., "Manually check Netflix careers at jobs.netflix.com (blocked by anti-bot)"}
  - [ ] {action 3 — e.g., "Query remaining 9 Greenhouse APIs not covered this run: {list}"}
  - [ ] {action 4 — e.g., "Add {company} to Greenhouse list — they may have migrated to Greenhouse"}
```

**This report is NOT optional.** Even if 0 roles were found, print it — especially if 0 roles were found. A "0 results + LOW false negative risk" is reassuring. A "0 results + HIGH false negative risk" tells the user to investigate further.

---

## PHASE 8.7: Write Checkpoint JSON ← NEW in v16.11

**This phase creates a safety net.** Before attempting the sheet write (the most failure-prone step), save all scored data to a checkpoint file. If Phase 9 fails, this file lets you retry just the write without re-running the entire pipeline.

Save to `{workspace}/scored_run{N}.json` with this structure:

```json
{
  "run_number": N,
  "date": "M/D/YYYY",
  "mode": "DISCOVERY",
  "input_candidates": 12,
  "jds_read": 10,
  "scored_roles": [
    {
      "id": "company_title_id",
      "title": "...",
      "company": "...",
      "team": "...",
      "location": "...",
      "url": "...",
      "fit": 72,
      "opportunity": 63,
      "interview_prob": 58,
      "action_gate": "NETWORK FIRST",
      "bucket": "B",
      "csv_row": [/* the 27-value array from Phase 6F */],
      "url_liveness": "LIVE_JD",
      "jd_hash": "..."
    }
  ],
  "skipped_roles": [
    {
      "id": "...",
      "title": "...",
      "company": "...",
      "skip_reason": "...",
      "fit": 42
    }
  ],
  "action_summary": {
    "apply_now": 0,
    "network_first": 1,
    "hold": 1,
    "hold_borderline": 5,
    "skip": 5
  }
}
```

The key feature is that `csv_row` arrays are embedded directly in each scored role — Phase 9 reads them straight from this file rather than reconstructing from memory. This is what makes the sheet write reliable even under context pressure.

**Fallback path**: If Phase 9 fails after retry, tell the user: "The scored data is saved at `scored_run{N}.json`. You can retry the sheet write anytime by saying 'sync run {N}' — the job-sync skill will pick up from this checkpoint." This provides a recovery path without losing any work.

---

## PHASE 9: Append to Google Sheet ← STRICT COLUMN MAPPING

**STOP: Read `references/output-formats.md` § Column-to-Array Position Mapping NOW.** This defines the exact 27-value array format. The reason this explicit mapping exists is that Run 16's Apps Script had a missing Location column and extra text fields, which shifted all data and corrupted 4 rows. The mapping table and validation code in the template prevent this.

**Source data**: Read csv_row arrays from `{workspace}/scored_run{N}.json` (written in Phase 8.7). Do NOT reconstruct the 27-value arrays from memory — the checkpoint file is the single source of truth for the sheet write. This is the key reliability improvement in v16.11: by the time Phase 9 runs, the data is already validated and formatted, so the write phase just needs to inject it.

Apps Script via Monaco API (see `references/output-formats.md`). Verify. Retry once. If retry fails → tell user about job-sync fallback.

### Pre-Write Column Validation (MANDATORY)

Before injecting the Apps Script, verify the data array for EACH row (these checks should pass trivially since the arrays were validated in Phase 6F, but defense-in-depth matters):
1. **Count**: Exactly 27 values per row (not 25, not 26)
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

Print this checklist and check off each field as you update it. The reason this explicit checklist exists is that Run 13's evaluation found 5 preference fields were silently skipped — `channel_performance`, `cluster_outcomes`, `weekly_kpis`, `referral_tracking`, and `sourcing_performance` — partially breaking the learning loop for subsequent runs. Run 16's evaluation found that `title_expansion_map`, `url_pattern_registry`, `linkedin_company_filters`, and `Google.application_budget` were silently skipped.

**MANDATORY**: Every field below MUST be explicitly addressed — either updated OR marked "no change — {reason}". Silent skips are NOT acceptable. When the checklist is printed at the end of Phase 10, every line MUST show either [✓] with the change made, or [—] with "no change — {reason}".

```
PHASE 10 UPDATE CHECKLIST:
[ ] last_run → "{date} (run {N})"
[ ] run_counter → {N}
[ ] company_search_log → Update times_searched + new_roles_found_last for EACH company searched
[ ] company_intelligence → Update EXISTING keys only (NEVER create "Company_intel" duplicates)
[ ] company_intelligence.title_vocabulary → Update from adaptive keyword expansion discoveries (v16.7)
[ ] company_intelligence.team_status → Update team-level freeze data if partial_freeze detected (v16.7)
[ ] title_expansion_map → Propose new adjacent titles if discovered this run (v16.11)
[ ] keyword_wisdom → Update times_used, avg_score, confidence for each keyword used this run
[ ] market_health → url_404_rate_by_run, trend, dead_channels_this_run, recommendation
[ ] learning_history.strategy_changes → Append run summary
[ ] learning_history.weekly_kpis → Add new entry if 7+ days since last snapshot
[ ] referral_tracking → Add entries for NEW networking targets (Moderate/Strong paths found this run)
[ ] sourcing_performance → Update if new applications submitted; note "no change" if none
[ ] channel_performance → Update if new Applied/Interview outcomes; note "no change" if none
[ ] cluster_outcomes → Update if new applications submitted; note "no change" if none
[ ] last_sheet_write_status → "success_run{N}_{X}rows_at_{Y}" or "failed_{reason}"
--- ATS REGISTRY (NEW in v16.2) ---
[ ] ats_registry → Update last_queried + filtered_roles_last for each company queried
[ ] ats_registry → Add newly probed companies (ats_type, slug, api_works, last_probed)
[ ] ats_registry → Flag failed APIs (api_works = false + note with error)
[ ] ats_registry → Re-probe stale entries (last_probed > 30 days ago)
--- PLATFORM MIGRATION TRACKING (NEW in v16.6) ---
[ ] url_pattern_registry → Compare this run's URL patterns against stored patterns (see below)
[ ] url_pattern_registry → Log any PLATFORM_MIGRATION flags from Phase 3.6
[ ] url_pattern_registry → Update career-sites.md reference if migration confirmed
[ ] linkedin_company_filters → Add newly discovered company IDs (see references/linkedin-company-filters.md)
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
--- v16.11 FIELDS (NEW — Run 16 evaluation found these were silently skipped) ---
[ ] company_intelligence.Google.application_budget → Update slots_used/available if Google roles applied to
[ ] url_pattern_registry → Add URL patterns for NEW companies scored this run (Reddit, Stripe, etc.)
[ ] Phase 2B keyword_matrix artifact → Save to {workspace}/keyword_matrix_run{N}.md (see Fix #5)
All fields checked: [ ] YES — Every field shows [✓] or [—] with reason
```

**Clarification on "no change" fields**: Not every field changes every run. If no new applications were submitted this run, `sourcing_performance`, `channel_performance`, and `cluster_outcomes` won't change — but you still need to acknowledge them ("no change — 0 new apps this run") rather than silently skipping. This prevents the ambiguity of "was this field skipped or was it unchanged?"

### Platform Migration Detection ← NEW in v16.6

**Why this exists**: Run 11 discovered that Microsoft migrated from `jobs.careers.microsoft.com` to `apply.careers.microsoft.com`, orphaning all existing job IDs (7-digit format completely dead on new platform). This wasn't detected until Phase 7.5, after the pipeline had already scored and ranked those roles. Platform migrations break ALL URLs for a company simultaneously — detecting them proactively prevents cascading failures.

**How it works — URL pattern comparison across runs:**

1. **Build this run's URL pattern fingerprint** — for each company that had roles in this run, record:
   - Career site domain (e.g., `apply.careers.microsoft.com`)
   - Job ID format (e.g., 16-digit PID vs 7-digit legacy ID)
   - URL structure (e.g., `/careers/job/{pid}` vs `/global/en/job/{id}`)

2. **Compare against stored patterns** in `preferences.url_pattern_registry`:
   ```json
   "url_pattern_registry": {
     "Microsoft": {
       "domain": "apply.careers.microsoft.com",
       "id_format": "16-digit PID",
       "url_template": "/careers/job/{id}",
       "last_verified": "2026-03-13",
       "previous_domain": "jobs.careers.microsoft.com",
       "migration_date": "2026-01"
     }
   }
   ```

3. **Flag mismatches**:
   - Domain changed → `PLATFORM_MIGRATION` — update registry, update `references/career-sites.md`
   - Job ID format changed → `ID_FORMAT_CHANGE` — all cached IDs for this company are suspect
   - URL structure changed → `URL_RESTRUCTURE` — existing URLs may still work but monitor

4. **Also check for Phase 3.6 migration flags**: If Phase 3.6 detected any `PLATFORM_MIGRATION` signals during early URL sampling, record them here.

5. **Print migration report** (only if changes detected):
   ```
   PLATFORM MIGRATION DETECTION:
     Microsoft: MIGRATION CONFIRMED — jobs.careers.microsoft.com → apply.careers.microsoft.com
       Old ID format: 7-digit (e.g., 1761259) — ALL DEAD on new platform
       New ID format: 16-digit PID (e.g., 1970393556826666)
       Action: Updated career-sites.md, url_pattern_registry
     {other companies if any}
   ```

6. **If no `url_pattern_registry` exists in preferences** (first run with this feature): Initialize it from this run's URL data. No comparison needed — just set the baseline.

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

**QUICK_TRIAGE**: Max 3 companies, page 1, IP-only, Ch1-2, skip signals/reasoning/tailoring/sheet.
**FAST_TRACK**: Max 5 companies, 2-3 pages, Fit+Opp(3)+IP, all 4 Ch (Ch4 lighter), light signals, top 3 reasoning, sheet append.
**FULL_ANALYSIS**: Unlimited, all pages, full 5-pillar, all 4 Ch full depth, full signals/reasoning/guardrails/tailoring, sheet + formatting.
**CRM and Interview Pipeline run in ALL modes** — never skipped.

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
