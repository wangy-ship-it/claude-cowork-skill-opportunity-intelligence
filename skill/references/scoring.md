# Scoring System — Three Scores + Action Gate

## Prerequisites

Before scoring, ensure:
1. Signal extraction is complete for this role (see `references/signal_extraction.md`)
2. Scoring operates on **extracted signals**, not raw JD text
3. Networking path strength has been evaluated (see `references/networking_strategy.md`)

## Fit Score (0-100) — Structured Rubric

Use Enhanced Contextual Matching — don't reduce to keyword overlap. Score each dimension using the **extracted signals** from signal_extraction.md.

| Dimension | Max | What to evaluate | Signal source |
|-----------|-----|-----------------|---------------|
| **Level match** | 25 | Scope, people management, org ownership. Use trajectory_type. | Role signals: seniority, leadership_scope, ic_vs_mgmt |
| **Function match** | 25 | Product analytics vs applied ML vs GTM vs data eng vs research. Decode JD jargon. | Domain signals: primary_domain, secondary_domains |
| **Core skills match** | 25 | Match against semantic_skill_clusters, not just exact tool names. | Technology signals: direct/transferable/gap counts |
| **Domain transferability** | 10 | Use transferable_patterns. Max 5-8 penalty for domain mismatch alone. | Domain signals + candidate transferable_patterns |
| **GenAI relevance** | 5 | Bonus if JD mentions GenAI/LLM. Check "nice to have." Zero penalty if not mentioned. | Technology signals: GenAI stack |
| **Recency** | 5 | +5 if ≤14 days; +3 if 15-30; +1 if 31-60; 0 if older | Posting date |
| **JD clarity** | 5 | Well-written with clear requirements? | Signal completeness |

**Labels**: 85+ = EXCELLENT MATCH, 70-84 = STRONG/GOOD MATCH, 55-69 = MODERATE MATCH, <55 = don't include

## Opportunity Score (0-100) — Five-Pillar Career-Strategic Value

| Pillar | Weight | Description |
|--------|--------|-------------|
| Culture Fit | 15% | Mission, culture reviews, org structure |
| Compensation & Growth | 25% | Base, equity, growth path |
| Role Leverage | 20% | Brand, network, new skills |
| Speed to Hire | 20% | Time to offer, rounds |
| ATS & Recruiter Friction | 20% | Resume match, complexity |

**FAST_TRACK**: 3 pillars — Compensation (35%), Leverage (35%), Speed (30%)
**QUICK_TRIAGE**: Skip entirely

### Detailed Rubrics (each 0-100)
- **Culture Fit**: 90+ perfect alignment; 75-89 strong; 60-74 decent; 40-59 weak; <40 poor
- **Compensation**: 90+ ≥85th percentile + equity; 75-89 = 75-85th; 60-74 market; 40-59 below; <40 poor
- **Role Leverage**: 90+ FAANG/Unicorn + 3 new skills; 75-89 growth-stage; 60-74 profitable private; 40-59 niche; <40 declining
- **Speed to Hire**: 90+ 1-2 weeks; 75-89 2-3 weeks; 60-74 3-6 weeks; 40-59 6-12 weeks; <40 3+ months
- **ATS Friction**: 90+ frictionless; 75-89 easy; 60-74 standard; 40-59 difficult; <40 black hole

## Interview Probability (0-100) — Conversion Likelihood

Predicts actual callback. Independent of Fit and Opportunity.

| Signal | Impact |
|--------|--------|
| JD keywords match historical interview-getters | +10-15 |
| Posted within 14 days | +10 |
| Target geo or remote-US | +5 |
| Scope matches candidate's recent scope | +10 |
| Clear JD with specific must-haves met | +10 |
| Hiring velocity (many similar roles) | +10 |
| Bucket A or C | +5-10 |
| No hard blockers | +10 |
| Historical resonance | +10-15 |
| Historical rejection pattern | -10-15 |
| Funnel openness | +0-10 |
| ATS friction (Workday -2, Greenhouse +3, Taleo -5, Ashby +2) | -5 to +3 |
| Black hole (≥5 apps, 0 screens) | capped at 55 |
| **Strong networking path** | **+20** |
| **Moderate networking path** | **+10** |
| Recruiter warm outreach | +15 |
| LinkedIn recruiter/HM post | +15 |
| Cold Bucket B (no referral, ≥5 data 0%) | capped at 55 |
| Company intelligence modifier | -15 to +10 |
| **Channel performance modifier** | -5 to +10 (from learning_loop.md) |
| **Opportunity cluster modifier** | -5 to +5 (from cluster learning) |

### Time-Based Decay (after base score)
- 0-14 days: no adjustment
- 15-30 days: no penalty
- 31-60 days: -10
- 61-90 days: -20
- 90+ days: -25
- 120+ days: auto-SKIP (zombie posting)

### Momentum Boost Stacking
| Signal | Boost |
|--------|-------|
| Recruiter reached out | +15 |
| Referral submitted (7 days) | +15 |
| HM directly engaged | +20 |
| Competitor mid-stage interview | +10 |
| Company hired from candidate's employer | +5 |
| LinkedIn HM post source | +15 |
| High growth intelligence | +10 |
| Layoff/freeze intelligence | -15 |

Compounds. Cap at 99.

### Tie-Break: IP → Bucket (A>C>B) → Fit → Opportunity

## Uplevel / Downlevel Risk

| Flag | HIGH | MED | LOW |
|------|------|-----|-----|
| Uplevel | Managing managers/VP/3+ teams | Director when SM | Good stretch |
| Downlevel | IC-only for SM+ | One level below | Appropriate |

## Action Gate — APPLY NOW / NETWORK FIRST / HOLD / SKIP

| Action | Criteria |
|--------|---------|
| **APPLY NOW** | Strong networking path AND IP ≥ 60 AND Fit ≥ 70 AND Opp ≥ 65 |
| **APPLY NOW (cold)** | No warm BUT Bucket A AND Fit ≥ 85 AND IP ≥ 75 AND Opp ≥ 70 |
| **NETWORK FIRST** | Moderate/Weak path AND Fit ≥ 70 AND Opp ≥ 65 AND IP ≥ 60 |
| **HOLD** | IP 40-59 OR Fit 65-69 OR Opp 55-64 |
| **SKIP** | IP < 40 OR Fit < 65 OR Opp < 55 OR blockers |

**Warm Channel Override**: All cold Bucket B → NETWORK FIRST. Override only if IP ≥ 80 AND Fit ≥ 85.
**Cooldown check**: Check `application_cooldowns` before APPLY NOW.
**Guardrails check**: Verify against `references/guardrails.md` constraints before finalizing.

### Reasoning Requirement
Each Action Gate assignment MUST include a reasoning block (see `references/decision_engine.md`).

### Adaptive Thresholds
- If <5 APPLY NOW → lower by 3, re-evaluate. Repeat once.
- **Hard floors**: Fit 65, IP 50, Opp 55
- If still <5 at floor → accept count, **trigger company expansion**
- If >10 → raise by 3, demote overflow

### Automatic Company Expansion
Trigger: <3 APPLY NOW AND <5 total includable.
1. Report diagnosis
2. Auto-discover 2-3 new companies
3. Search immediately
4. Present results, update preferences
5. Max 3 per expansion

## Bucket Assignment & Quota (50A / 30C / 20B)
Classify by company characteristics. Rebalance if significantly off-target (demote weakest B, promote top A/C HOLDs).

### Quota Enforcement Order
1. Extract signals → 2. Score all → 3. Black hole penalty → 4. Momentum boosts → 5. Time decay → 6. Networking path strength → 7. Assign buckets → 8. Action gates → 9. Guardrails check → 10. Adaptive thresholds → 11. Cap at 10 APPLY NOW → 12. Bucket quotas within 10

## Opportunity Clustering

Tag each role with an opportunity cluster based on extracted domain signals. Clusters group similar roles for strategic analysis and historical learning.

### Defined Clusters

| Cluster | Description | Detection |
|---------|-------------|-----------|
| **Product Data Science** | User-facing product analytics, experimentation, engagement metrics | primary_domain = product_data_science |
| **AI Platform Analytics** | ML infrastructure, model monitoring, platform metrics | primary_domain = ai_ml_platform |
| **Applied AI** | GenAI, LLM applications, NLP, recommendation systems | primary_domain = applied_ai |
| **Experimentation Systems** | A/B testing platforms, causal inference infrastructure | primary_domain = experimentation |
| **Data Platform Leadership** | Data engineering management, warehouse/pipeline leadership | primary_domain = data_platform_leadership |
| **Growth Analytics** | Growth, monetization, marketplace, pricing, retention | primary_domain = growth_analytics |
| **Security/SaaS Analytics** | Cybersecurity analytics, enterprise SaaS, B2B | primary_domain = security_saas |
| **Strategic Finance DS** | Financial analytics, strategy & ops, revenue forecasting | primary_domain = strategic_finance |

### How Clusters Affect the Pipeline

1. **Historical learning**: Track which clusters produce interviews (see `references/learning_loop.md` § Opportunity Cluster Learning). Clusters with higher interview rates get a +5 IP boost.

2. **Prioritization**: When multiple NETWORK FIRST roles compete for attention, prioritize roles in historically successful clusters.

3. **Strategy insight**: If 80% of interviews come from "Product Data Science" and "Growth Analytics" clusters, the Summary Report should highlight this and suggest focusing search keywords on those domains.

4. **Diversification check**: If all APPLY NOW roles are in a single cluster, note the concentration risk — a single domain downturn would eliminate all options.

### Cluster Assignment

Assign during signal extraction (Phase 4.5). Each role gets exactly one primary cluster based on the dominant domain signal. Store in the signal block and carry through to CSV and Google Sheet for tracking.

## Apply Accumulated Learning
Apply pre-computed `learning_history` from preferences:
1. `interview_prob_boost_keywords` (cap +20)
2. `interview_prob_penalty_keywords` (cap -20)
3. `company_modifiers`
4. `black_hole_companies` cap
5. Bucket conversion weighting (±3 to ±5)
6. Hot patterns (+5/+10/+15 by signal_count 2/3-4/5+)
7. Cold patterns (-5/-10/-15 by signal_count)
8. **Channel performance modifiers** (from learning_loop.md)
9. **Opportunity cluster modifiers** (from cluster learning data)

## Energy Budget
If APPLY NOW > 7 AND ≥3 pending today → stop early, output top 7, demote rest. User can override.

## How Scores Influence Networking and Pipeline (v14)

The three scores (Fit, Opportunity, Interview Probability) now influence not just the Action Gate but also networking and pipeline management:

### Networking Priority
When multiple NETWORK FIRST roles compete for outreach time, prioritize by:
1. Roles in historically successful clusters (highest interview conversion rate)
2. Roles with highest Interview Probability (after networking boost)
3. Roles with shortest expected time-to-convert (Moderate paths before Weak)

### Outreach Targeting
The scoring system informs outreach strategy:
- High Fit + High Opportunity + Low IP → networking is the bottleneck, invest heavily in finding contacts
- High Fit + Low Opportunity → may not be worth networking effort; HOLD unless easy warm path exists
- Low Fit + High Opportunity → skip networking, not a good match regardless of channel

### Pipeline Focus
Scores determine how much interview prep effort to invest:
- APPLY NOW roles with Fit ≥ 85 → full prep investment
- NETWORK FIRST roles converting to interview → prioritize prep based on Opportunity Score
- Roles where Fit is borderline (65-70) → prep should emphasize bridging the gap areas identified in signal extraction

## Opportunity Cluster Success Metrics (v14)

Track success metrics per cluster to enable strategic prioritization:

```json
{
  "cluster_metrics": {
    "product_data_science": {
      "applications": 8,
      "interviews": 3,
      "offers": 1,
      "interview_rate": 0.375,
      "offer_rate": 0.125,
      "avg_fit_score": 82,
      "avg_days_to_interview": 14,
      "primary_channels_used": {"referral": 3, "cold_apply": 5}
    }
  }
}
```

When ≥5 data points exist per cluster:
- Clusters with interview_rate > 30% → +5 IP boost AND prioritize in networking queue
- Clusters with interview_rate > 50% → +10 IP boost (strong signal of market fit)
- Clusters with 0% interview_rate on ≥5 apps → -5 IP penalty AND suggest pivoting search keywords
- Surface top 2 clusters in Daily Briefing as "Focus Areas"
- If >80% of interviews come from 1-2 clusters → highlight this in strategy recommendations
