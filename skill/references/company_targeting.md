# Company Targeting Engine — Phase 0.5 Reference

> **Purpose**: Replace the static, ever-growing company list with an intelligent targeting system that discovers, categorizes, rotates, and prunes companies based on profile fit, historical yield, and market signals.

## Table of Contents
1. [Company Targeting Config Schema](#schema)
2. [Cadence Tiers](#cadence-tiers)
3. [Industry Tags](#industry-tags)
4. [Intelligent Company Discovery](#discovery)
5. [Yield Tracking](#yield-tracking)
6. [Phase 0.5 Execution Steps](#execution)
7. [Migration from Legacy System](#migration)
8. [Quick-Reject Gate](#quick-reject)

---

## Company Targeting Config Schema {#schema}

Lives in `preferences.company_targeting`:

```json
{
  "company_targeting": {
    "version": "1.0",
    "last_optimized": "date",

    "cadence_tiers": {
      "every_run": [],
      "every_other": [],
      "weekly": [],
      "monthly": [],
      "networking_only": [],
      "dormant": []
    },

    "industry_tags": {
      "CompanyName": ["ecommerce", "marketplace"]
    },

    "profile_fit_scores": {
      "CompanyName": {
        "industry_fit": "high|medium|low",
        "function_match": "analytics|ds|ml_eng|mixed",
        "level_match": "exact|adjacent|mismatch",
        "last_yield": 0,
        "consecutive_zeros": 0,
        "lifetime_yield": 0,
        "lifetime_scored_above_70": 0,
        "last_action_gates": []
      }
    },

    "discovery_queue": [],

    "auto_rules": {
      "promote_after_interview": true,
      "demote_after_consecutive_zeros": 5,
      "dormant_after_consecutive_zeros": 8,
      "max_every_run": 25,
      "max_per_run_total": 40,
      "probe_new_companies": true,
      "discovery_budget_per_run": { "FULL_ANALYSIS": 5, "FAST_TRACK": 2, "QUICK_TRIAGE": 0 }
    }
  }
}
```

---

## Cadence Tiers {#cadence-tiers}

Companies rotate through tiers based on yield and strategic value. The tier determines how often a company appears in the search queue.

| Tier | Search Frequency | Criteria | Max Size |
|------|-----------------|----------|----------|
| **every_run** | Every run | Active pipeline, interview history, referral contacts, proven yield (≥2 scored roles in last 3 runs) | 25 |
| **every_other** | Every 2nd run | Moderate yield (1 scored role in last 3 runs) OR strategic target with working ATS API | 30 |
| **weekly** | ~Every 3rd run | Low yield but profile-relevant industry. Includes most ATS API companies | 40 |
| **monthly** | ~Every 8th run | Chronic zero-yielders that still have ATS API access | unlimited |
| **networking_only** | Never searched | Companies with CRM contacts but no role match. Never searched in Ch1-3, but CRM stays active. If a matching role surfaces externally, auto-promotes to every_run | unlimited |
| **dormant** | Never searched | Confirmed function mismatch, dealbreakers, black holes | unlimited |

### Auto-Promotion Rules

**Promote to every_run when:**
- Company gave an interview (check `learning_history.conversion_rates.by_company`)
- Company has an active referral with `outreach_status` in [referral_offered, referral_submitted]
- Company had ≥2 roles scoring Fit ≥70 in the last 3 runs
- A `networking_only` company gets a matching role sent by a contact → auto-promote

**Promote to every_other when:**
- Company had 1 role scoring Fit ≥70 in the last 3 runs
- Company is in ATS registry with `api_works = true` AND `filtered_roles_last ≥ 1`

### Auto-Demotion Rules

**Demote to monthly when:**
- `consecutive_zeros ≥ 5` (5+ runs with 0 relevant roles after title filtering)
- Exception: `networking_only` companies are never demoted — they're already correctly placed

**Demote to dormant when:**
- `consecutive_zeros ≥ 8` AND `lifetime_scored_above_70 == 0`
- Company is in `candidate_profile.company_dealbreakers`
- Company is in `learning_history.black_hole_companies`

### Special Case: Apple-Type Companies

Some companies are strategic referral targets that rarely post matching roles. These belong in `networking_only`:
- The user has contacts who can refer them
- The company is large enough that roles appear sporadically
- Searching every run wastes context, but abandoning the company loses referral potential
- When Apple does post "Senior Manager, Data Analytics", the CRM contact can flag it and the company auto-promotes

---

## Industry Tags {#industry-tags}

Every company gets one or more industry tags. These drive discovery, portfolio balance, and function-match prediction.

### Tag Taxonomy

| Tag | Description | Analytics Role Density |
|-----|-------------|----------------------|
| `ecommerce` | Online retail, DTC brands | HIGH |
| `marketplace` | Two-sided platforms | HIGH |
| `fintech` | Financial technology, lending | HIGH |
| `payments` | Payment processing | MEDIUM |
| `adtech` | Advertising, measurement | HIGH |
| `consumer_social` | Social media, content | MEDIUM |
| `enterprise_saas` | B2B software | MEDIUM |
| `healthtech` | Healthcare technology | MEDIUM |
| `edtech` | Education technology | MEDIUM |
| `travel` | Travel, hospitality, mobility | HIGH |
| `food_delivery` | Food/grocery delivery | HIGH |
| `gaming` | Gaming, entertainment | MEDIUM |
| `security` | Cybersecurity, defense | LOW |
| `infra` | Dev tools, cloud, data infra | LOW |
| `ai_native` | AI-first companies | LOW-MEDIUM |
| `media` | Media, publishing | MEDIUM |
| `retail` | Physical + digital retail | HIGH |
| `financial_services` | Banks, insurance, trad finance | HIGH |

### Function Match Prediction

Analytics Role Density predicts what fraction of a company's data roles match an analytics/DS leadership profile vs. ML engineering:

- **HIGH** (>60% analytics): ecommerce, marketplace, fintech, adtech, travel, food_delivery, retail, financial_services → prioritize for company discovery
- **MEDIUM** (30-60%): consumer_social, enterprise_saas, healthtech, edtech, gaming, media → accept some SKIP rate
- **LOW** (<30% analytics): security, infra, ai_native → limit to weekly/monthly cadence unless specific role signal

### Mapping Candidate Domains to Industries

Use the candidate's `domain_expertise` to identify high-priority industries for discovery:

```
customer analytics       → ecommerce, marketplace, fintech, retail, consumer_social
retention/churn modeling → consumer_social, enterprise_saas, edtech, fintech
GTM strategy            → enterprise_saas, adtech, fintech
marketplace optimization → marketplace, food_delivery, travel
e-commerce analytics    → ecommerce, retail, marketplace
GenAI adoption          → ai_native, enterprise_saas (applied AI teams)
```

---

## Intelligent Company Discovery {#discovery}

Instead of manually maintaining a target list, the engine proposes new companies each run via four sources.

### Source 1: Profile-Based Expansion

For each of the candidate's domain expertise areas:
1. Map to HIGH-density industry tags (see above)
2. Search for well-known companies in those industries that aren't yet in the queue
3. Filter by: 500+ employees, offices in target locations or remote-friendly
4. Check ATS registry — if the company has a known working API, it's a stronger candidate

This is the primary discovery mechanism. It answers: "What companies would naturally hire someone with this profile?"

### Source 2: Competitor Expansion

When a company produces a scored role with Fit ≥65:
1. Identify 2-3 direct competitors in the same industry
2. If they're not in the queue, propose them
3. Rationale: If Company A needs analytics leadership, competitors B and C probably do too

### Source 3: Referral Network

When a new CRM contact is added:
1. Check their current company — is it in the queue?
2. Their current and former companies are natural candidates
3. People in the same industry tend to cluster at similar companies

### Source 4: Trending / News-Based

Capture signals during Phase 2D (Company Intelligence):
- Companies announcing analytics team expansions
- Companies that recently raised Series C+
- Companies posting multiple analytics leadership roles
- Add these to `discovery_queue` for next run's probing

### Discovery Budget

| Mode | New Companies Per Run |
|------|----------------------|
| FULL_ANALYSIS | Up to 5 |
| FAST_TRACK | Up to 2 |
| QUICK_TRIAGE | 0 |

For each proposed company, run ATS Probe (from Phase 3 protocol) before adding. If the probe finds a working API with matching roles, that's a strong signal.

### Discovery Output Template

```
═══════════════════════════════════════════════════
COMPANY QUEUE OPTIMIZATION — {date}
═══════════════════════════════════════════════════

NEW COMPANIES DISCOVERED:
  🆕 {Company} ({industry_tags})
     Source: {profile_based|competitor|referral|trending}
     Why: {reasoning}
     ATS: {probe_result}
     → Assigned to: {cadence_tier}

CADENCE CHANGES THIS RUN:
  ⬆️ {Company}: {old_tier} → {new_tier} ({reason})
  ⬇️ {Company}: {old_tier} → {new_tier} ({reason})

PORTFOLIO BALANCE:
  {industry}: {count} companies ({pct}% of active queue)
  ...
  ⚠ Underweight: {industry} — suggest {companies}
  ⚠ Overweight: {industry} — consider demoting chronic zeros

THIS RUN'S SEARCH QUEUE ({total} companies):
  every_run: {list}
  every_other (this run): {list}
  weekly (this run): {list}
  discovery (new): {list}
```

---

## Yield Tracking {#yield-tracking}

Each company's `profile_fit_scores` entry tracks yield metrics that drive cadence decisions:

- `last_yield`: Roles found in most recent search
- `consecutive_zeros`: How many runs in a row with 0 relevant roles
- `lifetime_yield`: Total roles ever found at this company
- `lifetime_scored_above_70`: Roles that scored Fit ≥70 — indicates genuine profile match
- `last_action_gates`: Action gates assigned to last batch of roles (e.g., ["SKIP", "SKIP", "HOLD"])
- `function_match`: Derived from what roles the company posts — "analytics" if >60% match, "ml_eng" if <30% match, "mixed" otherwise

**Function breakdown matters more than raw yield.** Pinterest yields 5 roles per run, but they're all ML engineering — it's a bad target for an analytics candidate. A company yielding 1 analytics leadership role is more valuable than one yielding 5 ML eng roles.

Update yield tracking in Phase 10 after every run.

---

## Phase 0.5 Execution Steps {#execution}

Runs after Phase 0 (load preferences) and before Phase 1 (daily check-in).

### Step 1: Load or Initialize

Load `preferences.company_targeting`. If missing (first run with this feature), run migration (see below).

### Step 2: Apply Auto-Promotion/Demotion

For each company:
1. Check interview history → promote to every_run if interview received
2. Check CRM contacts → promote if active referral exists
3. Check consecutive zeros → demote if threshold met
4. Check function breakdown → demote if >70% ML eng roles
5. Check dealbreakers/black holes → move to dormant

### Step 3: Run Company Discovery

Based on mode's discovery budget:
1. Profile-based expansion — find companies in HIGH-density industries not yet in queue
2. Competitor expansion — find competitors of recent high-Fit companies
3. Referral network — check new CRM contacts
4. Trending — check discovery_queue populated by previous Phase 2D

For each new company: run ATS Probe, assign industry tags, set initial tier (usually `every_other`).

### Step 4: Build This Run's Search Queue

Combine companies based on cadence tiers:
- ALL `every_run` companies
- Half of `every_other` (alternate using `run_counter % 2`)
- One-third of `weekly` (use `run_counter % 3`)
- `monthly` only if `run_counter % 8 == 0`
- All newly discovered companies (always searched first time)

Cap total at `auto_rules.max_per_run_total` (default 40).

### Step 5: Display Optimization Report

Print the discovery output template showing new proposals, cadence changes, portfolio balance, and this run's queue.

### Step 6: Save to Preferences

Update `company_targeting` with all changes before proceeding to Phase 1.

---

## Migration from Legacy System {#migration}

For users with `company_search_log` but no `company_targeting`:

### Step 1: Initialize Cadence Tiers

Classify each company from the search log:
- Companies with interviews → `every_run`
- Companies with active CRM contacts but low/zero yield → `networking_only`
- Companies with active CRM contacts AND yield > 0 → `every_run`
- Companies with `new_roles_found_last > 0` and historical Fit ≥65 → `every_other`
- Companies with `new_roles_found_last > 0` but low Fit → `weekly`
- Companies with 5+ consecutive zeros → `monthly`
- Companies with 8+ consecutive zeros and no Fit ≥70 ever → `dormant`
- Dealbreakers → `dormant`

### Step 2: Assign Industry Tags

One-time classification based on company name and known domain. Use the industry tag taxonomy. Most tech companies can be tagged based on their primary product/market.

### Step 3: Compute Profile Fit Scores

Use historical data from `company_search_log` and Google Sheet to populate yield tracking fields.

### Step 4: Seed Discovery Queue

Add analytics-heavy companies not yet in the search log. The seed list should emphasize HIGH-density industries matching the candidate's domain expertise.

---

## Quick-Reject Gate {#quick-reject}

Phase 0.5 enables a context-saving optimization for Phase 3: companies tagged with `function_match: "ml_eng"` get stricter title filtering. Before JD reading in Phase 4, reject any role with these titles immediately:

- "ML Engineer", "Machine Learning Engineer"
- "Applied ML", "ML Platform"
- "AI Engineer" (without "Analytics" or "Science" modifier)
- "Full Stack Engineer", "Backend Engineer", "Frontend Engineer"

These would always SKIP for an analytics leadership candidate. Rejecting them before JD reading saves significant context — in Run 9, this would have eliminated 7 of 18 JD reads.

The rejection is conservative: any title that includes "Analytics", "Data Science", "Insights", "Decision", or "Experimentation" alongside ML terms should still proceed to JD reading (it might be a hybrid role).
