# Learning Loop

This file defines how the pipeline learns from outcomes over time. Every run produces data. The learning loop converts that data into scoring adjustments, strategy shifts, and better predictions.

## Existing Learning (from Phase 1 Daily Check-in)

The pipeline already tracks:
- Conversion rates by company, bucket, ATS type, team type
- Hot patterns (what produces interviews)
- Cold patterns (what produces rejections)
- Black hole companies
- Score adjustments (keyword boosts/penalties, company modifiers)
- Sourcing performance by channel
- Weekly KPIs

This file adds **Channel Performance Tracking** and formalizes how learning feeds back into scoring.

## Channel Performance Tracking

Track outcomes for each engagement channel independently. This reveals which channels actually produce results for this specific candidate, not just general assumptions.

### Channels to Track

| Channel | Description | Example |
|---------|-------------|---------|
| **cold_apply** | Applied via career site or ATS with no warm connection | Submit resume on jobs.apple.com |
| **recruiter_outreach** | Recruiter reached out first (inbound) | LinkedIn recruiter message |
| **referral** | Applied with internal referral from known contact | Friend submitted referral at Meta |
| **hiring_manager_message** | Responded to HM's LinkedIn post or direct message | Replied to hiring manager's post |
| **alumni_connection** | Applied after warm intro via alumni network | UIUC alumni connection at Stripe |
| **second_degree_intro** | Applied after mutual connection introduction | Friend-of-friend intro at Google |

### Metrics Per Channel

For each channel, track:

```json
{
  "channel_name": {
    "applications": 0,
    "responses": 0,
    "interviews": 0,
    "offers": 0,
    "response_rate": 0.0,
    "interview_rate": 0.0,
    "avg_days_to_response": null,
    "last_used": null
  }
}
```

### How Channel Data Adjusts Scoring

After accumulating ≥3 data points for a channel:

1. **Interview Probability adjustment**: If a channel's `interview_rate` is significantly different from the baseline (±15%), adjust IP for roles where that channel would be used:
   - Channel interview_rate > 50% → +10 IP when that channel is available
   - Channel interview_rate 25-50% → +5 IP
   - Channel interview_rate 10-25% → no adjustment
   - Channel interview_rate < 10% with ≥5 data points → -5 IP
   - Channel interview_rate = 0% with ≥5 data points → cap IP at 55

2. **Action Gate influence**: If the candidate's best available channel for a role has historically poor conversion, downgrade the gate:
   - APPLY NOW → NETWORK FIRST (if the only channel is cold_apply with 0% interview rate on ≥5 applications)
   - This is the existing "warm channel override" formalized with data

3. **Strategy recommendations**: When channel data shows clear winners:
   - If referral > 50% and cold_apply < 5% → recommend "STOP cold applying to Bucket B, invest all energy in referrals"
   - If hiring_manager_message > 40% → recommend "Prioritize Channel 4 (recruiter/HM posts) for lead generation"
   - If recruiter_outreach > 60% → recommend "Build recruiter relationships, respond quickly to inbound"

### Updating Channel Data

During Phase 1 (Daily Check-in), when reading new Applied/Interview rows from the sheet:
1. Identify the channel used (from the Referral checkbox, Stage notes, or Next Action field)
2. Increment the appropriate channel counters
3. Recompute rates
4. Store in `sourcing_performance.by_channel` in preferences

### Channel Performance Display

Add to Daily Briefing:

```
CHANNEL PERFORMANCE:
  referral:          3 apps → 3 interviews (100%) ★ best channel
  recruiter_outreach: 2 apps → 2 interviews (100%) ★
  cold_apply:        17 apps → 0 interviews (0%) ⚠ worst channel
  hm_message:        0 apps (untested)
  alumni_connection:  0 apps (untested)

  INSIGHT: Warm channels 100% vs cold 0%. Every networking action > every cold app.
```

## Opportunity Cluster Learning

Track which **opportunity clusters** (see scoring.md § Opportunity Clustering) produce interviews:

```json
{
  "cluster_outcomes": {
    "product_data_science": {"applied": 5, "interviews": 2, "rate": 0.40},
    "growth_analytics": {"applied": 3, "interviews": 1, "rate": 0.33},
    "applied_ai": {"applied": 2, "interviews": 0, "rate": 0.00},
    "data_platform_leadership": {"applied": 3, "interviews": 0, "rate": 0.00}
  }
}
```

After ≥3 data points per cluster:
- Clusters with interview_rate > 30% → +5 IP boost for roles in that cluster
- Clusters with interview_rate = 0% and ≥5 applications → -5 IP penalty
- Surface in Daily Briefing as "Best-performing domains" and "Underperforming domains"

## Phase 10 Field-by-Field Update Guide

This section exists because Run 13's evaluation found 5 preference fields were silently skipped during Phase 10, partially breaking the learning loop. The pipeline was updating `last_run`, `run_counter`, and a few others, but quietly dropping `channel_performance`, `cluster_outcomes`, `weekly_kpis`, `referral_tracking`, and `sourcing_performance`. Since those fields feed directly into next-run scoring adjustments (channel → IP adjustment, cluster → IP boost/penalty, KPIs → trend detection), skipping them means the pipeline doesn't actually learn.

For each field below, here's what to update and when it's OK to leave unchanged:

| Field | What to update | OK to skip when |
|-------|---------------|-----------------|
| `last_run` | Always update to current date + run number | Never skip |
| `run_counter` | Always increment | Never skip |
| `company_search_log` | `times_searched`, `new_roles_found_last` for each company | Never skip |
| `company_intelligence` | Update EXISTING key with fresh intel. Never create `Company_intel` duplicates | No new intel and last check <7 days |
| `keyword_wisdom` | `times_used`, `avg_score`, `confidence` for each keyword | Never skip |
| `market_health` | `url_404_rate_by_run`, `trend`, `dead_channels_this_run` | Never skip |
| `learning_history.strategy_changes` | Append run summary entry | Never skip |
| `learning_history.weekly_kpis` | Add snapshot if 7+ days since last entry | Last entry <7 days old |
| `referral_tracking` | Add entries for NEW networking targets found this run | No new Moderate/Strong paths found |
| `sourcing_performance` | Update `by_channel` if new applications were submitted | 0 new applications this run — note "no change" |
| `channel_performance` | Update per-channel metrics if new outcomes detected | 0 new Applied/Interview changes — note "no change" |
| `cluster_outcomes` | Update per-cluster stats if new applications submitted | 0 new applications — note "no change" |
| `last_sheet_write_status` | Record success/failure of Google Sheet write | Never skip |

**The "no change" rule**: When a field genuinely hasn't changed (e.g., no new applications means `channel_performance` is unchanged), explicitly acknowledge it: "channel_performance: no change — 0 new apps this run." This distinguishes "intentionally unchanged" from "accidentally forgotten." Silent skipping is the bug this checklist prevents.

## Learning Feedback Loop Summary (v14)

```
Sheet data (Phase 1)
  → Channel performance update
  → Cluster outcome update
  → Keyword wisdom update
  → Company modifier update
  → Hot/cold pattern detection
  → Score adjustment recalculation
  → Strategy change proposals
CRM data (Phase 1.5)
  → Contact follow-up processing
  → Relationship strength transitions
  → Networking pipeline state updates
  → Referral effectiveness tracking
Interview data (Phase 1.7)
  → Interview stage updates
  → Prep task generation
  → Thank-you note tracking
  → Stage-specific rejection analysis
  → Updated preferences saved (Phase 10)
```

Every run gets smarter. No insight is lost between sessions.

## Channel Performance Enhanced by CRM Data (v14)

The CRM layer provides more accurate channel attribution. Instead of inferring channels from sheet data alone, the CRM records the actual channel used for each opportunity:

- If `opportunity_pipelines.{id}.contact_id` exists and the contact's `outreach_status` = `referral_submitted` → channel = `referral`
- If the contact's `connection_source` = `alumni` → channel = `alumni_connection`
- If the contact's `connection_source` = `hiring_manager` → channel = `hiring_manager_message`
- If no CRM contact linked → channel = `cold_apply`

This gives more granular channel performance data than the previous approach.

## Interview Stage Learning (v14)

Track rejection patterns by interview stage to identify preparation gaps:

```json
{
  "interview_stage_outcomes": {
    "phone_screen": {"passed": 5, "rejected": 2, "pass_rate": 0.71},
    "technical_screen": {"passed": 3, "rejected": 1, "pass_rate": 0.75},
    "onsite_round": {"passed": 1, "rejected": 2, "pass_rate": 0.33}
  }
}
```

When a stage shows consistently low pass rate (< 50% with ≥3 data points):
- Surface in Daily Briefing as an area for improvement
- Suggest targeted preparation for that stage type
- If onsite pass rate is low → recommend mock interviews or deeper STAR story prep
- If phone screen pass rate is low → recommend positioning/pitch practice

## Networking Effectiveness Learning (v14)

Track which networking approaches actually lead to referrals and interviews:

```json
{
  "networking_effectiveness": {
    "alumni_outreach": {"sent": 5, "responded": 3, "referrals": 2, "response_rate": 0.60},
    "second_degree_intro": {"sent": 3, "responded": 2, "referrals": 1, "response_rate": 0.67},
    "cold_content_engagement": {"sent": 4, "responded": 1, "referrals": 0, "response_rate": 0.25},
    "former_colleague": {"sent": 2, "responded": 2, "referrals": 2, "response_rate": 1.00}
  }
}
```

Use this data to:
- Prioritize outreach to connection sources with higher response/referral rates
- Recommend strategy shifts (e.g., "Alumni outreach converts better than content engagement — focus there")
- Adjust Interview Probability networking boosts based on actual effectiveness
