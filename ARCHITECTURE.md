# Architecture & Design Deep Dive

This document provides a comprehensive technical walkthrough of the Job Search AI Skill — its components, data flows, scoring algorithms, and design rationale.

---

## Table of Contents

- [System Overview](#system-overview)
- [Execution Pipeline](#execution-pipeline)
- [Multi-Channel Search Engine](#multi-channel-search-engine)
- [Signal Extraction Pipeline](#signal-extraction-pipeline)
- [Three-Score Scoring System](#three-score-scoring-system)
- [Action Gate Decision Engine](#action-gate-decision-engine)
- [Networking CRM State Machine](#networking-crm-state-machine)
- [Interview Pipeline](#interview-pipeline)
- [Learning Loop](#learning-loop)
- [State Management](#state-management)
- [Data Flow Diagram](#data-flow-diagram)
- [Module Dependency Graph](#module-dependency-graph)

---

## System Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     JOB SEARCH AI SKILL v15                             │
│                                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌──────────────────────────┐    │
│  │  OPPORTUNITY │    │ RELATIONSHIP│    │  INTERVIEW PIPELINE      │    │
│  │  DISCOVERY   │    │    CRM      │    │  MANAGEMENT              │    │
│  │             │    │             │    │                          │    │
│  │ • 4-channel │◄──►│ • Contacts  │◄──►│ • Stage tracking         │    │
│  │   search    │    │ • 8-state   │    │ • Prep task generation   │    │
│  │ • Signal    │    │   pipeline  │    │ • Thank-you management   │    │
│  │   extraction│    │ • Follow-up │    │ • Outcome recording      │    │
│  │ • Scoring   │    │   automation│    │                          │    │
│  │ • Gating    │    │ • Strength  │    │                          │    │
│  └──────┬──────┘    │   tracking  │    └────────────┬─────────────┘    │
│         │           └──────┬──────┘                 │                  │
│         │                  │                        │                  │
│         └──────────────────┼────────────────────────┘                  │
│                            │                                           │
│                   ┌────────┴────────┐                                  │
│                   │  LEARNING LOOP  │                                  │
│                   │                 │                                  │
│                   │ • Channel perf  │                                  │
│                   │ • Cluster learn │                                  │
│                   │ • Score adjust  │                                  │
│                   │ • Hot/cold track│                                  │
│                   └────────┬────────┘                                  │
│                            │                                           │
│                   ┌────────┴────────┐                                  │
│                   │  PERSISTENCE    │                                  │
│                   │                 │                                  │
│                   │ preferences.json│                                  │
│                   │ Google Sheets   │                                  │
│                   │ CSV exports     │                                  │
│                   └─────────────────┘                                  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Execution Pipeline

The system operates through a strict 10-phase pipeline. Each phase has defined inputs, outputs, and checkpoints.

```
Phase 0 ─── Load State ──────────────────────────────────────────────►
   │         Read preferences.json, validate schema
   ▼
Phase 1 ─── Daily Check-in ─────────────────────────────────────────►
   │         CRM follow-ups, interview pipeline, stage updates
   ▼
Phase 2 ─── Search Strategy ────────────────────────────────────────►
   │         2A: Build search queue (companies × keywords)
   │         2B: Keyword matrix with synonyms
   │         2C: Market health check
   │         2D: Company intelligence refresh
   ▼
Phase 3 ─── Multi-Channel Search ───────────────────────────────────►
   │         Ch1: Career sites (agent)
   │         Ch2: LinkedIn Jobs (browser)
   │         Ch3: Boolean/ATS boards (agent)
   │         Ch4: LinkedIn recruiter posts (browser)
   ▼
Phase 4 ─── Read JDs ──────────────────────────────────────────────►
   │         Full JD text extraction via browser + agents
   │
   ├── 4.5 ── Signal Extraction ─────────────────────────────────►
   │           Role / Domain / Technology / Company signals
   ▼
Phase 5 ─── Score & Classify ───────────────────────────────────────►
   │         Fit Score + Opportunity Score + Interview Probability
   │         Action Gate assignment (APPLY NOW / NETWORK FIRST / HOLD / SKIP)
   │
   ├── 5.5 ── Networking Path Evaluation ────────────────────────►
   │           Strong / Moderate / Weak path classification
   ▼
Phase 6 ─── Action Pack Generation ─────────────────────────────────►
   │         Per-role execution plans, outreach drafts, resume bullets
   ▼
Phase 7 ─── CSV Export ─────────────────────────────────────────────►
   │         26-column standardized format
   ▼
Phase 8 ─── Summary Report ────────────────────────────────────────►
   │         Human-readable run summary with key metrics
   ▼
Phase 9 ─── Google Sheet Write ─────────────────────────────────────►
   │         Apps Script injection → Sheet append
   ▼
Phase 10 ── Save Preferences ───────────────────────────────────────►
             Update all learning fields, CRM, counters
```

### Phase Dependencies

```
                    Phase 0 (Load State)
                         │
                    Phase 1 (Daily Check-in)
                         │
                    Phase 2 (Strategy)
                    ┌────┴────┐
                    │  2A-2D  │  (sub-phases run sequentially)
                    └────┬────┘
                         │
                    Phase 3 (Search)
              ┌─────┬────┴────┬─────┐
              Ch1   Ch2       Ch3   Ch4    ← Channels run in parallel
              └─────┴────┬────┴─────┘
                         │
                    Phase 4 (Read JDs)
                         │
                    Phase 4.5 (Signals)    ← REQUIRED before scoring
                         │
                    Phase 5 (Score)
                         │
                    Phase 5.5 (Network Eval)
                         │
              ┌──────────┼──────────┐
              Phase 6    Phase 7    Phase 8    ← Can run in parallel
              └──────────┼──────────┘
                         │
                    Phase 9 (Sheet Write)
                         │
                    Phase 10 (Save)
```

---

## Multi-Channel Search Engine

Four complementary channels maximize coverage with different strengths:

```
┌──────────────────────────────────────────────────────────────────┐
│                    CHANNEL ARCHITECTURE                           │
│                                                                   │
│  ┌─────────────────┐    ┌─────────────────────┐                  │
│  │ Ch1: Career Sites│    │ Ch2: LinkedIn Jobs   │                 │
│  │                  │    │                      │                 │
│  │ Method:          │    │ Method:              │                 │
│  │  Web search +    │    │  Browser automation  │                 │
│  │  Agent scraping  │    │  JavaScript extract  │                 │
│  │                  │    │                      │                 │
│  │ Strengths:       │    │ Strengths:           │                 │
│  │  • Fresh data    │    │  • Broad coverage    │                 │
│  │  • Direct links  │    │  • Filter by date    │                 │
│  │  • Company-      │    │  • Salary data       │                 │
│  │    specific      │    │  • Easy apply flag   │                 │
│  │                  │    │                      │                 │
│  │ ATS Patterns:    │    │ Extraction:          │                 │
│  │  Greenhouse API  │    │  CSS selectors on    │                 │
│  │  Lever API       │    │  jobs-search-results │                 │
│  │  Ashby API       │    │  + JD content panel  │                 │
│  │  Workday search  │    │                      │                 │
│  └─────────────────┘    └─────────────────────┘                  │
│                                                                   │
│  ┌─────────────────┐    ┌─────────────────────┐                  │
│  │ Ch3: Boolean/ATS│    │ Ch4: LinkedIn Posts   │                 │
│  │                  │    │                      │                 │
│  │ Method:          │    │ Method:              │                 │
│  │  Direct ATS      │    │  Browser + a11y tree │                 │
│  │  API queries     │    │  Content search      │                 │
│  │                  │    │                      │                 │
│  │ Strengths:       │    │ Strengths:           │                 │
│  │  • Hidden roles  │    │  • Warm leads        │                 │
│  │  • Pre-posting   │    │  • HM direct access  │                 │
│  │  • Structured    │    │  • Team culture cues │                 │
│  │    data          │    │  • Hiring urgency    │                 │
│  │                  │    │    signals           │                 │
│  │ Query Pattern:   │    │                      │                 │
│  │  /api/v1/boards/ │    │ Note: Uses find +    │                 │
│  │  {board}/jobs?   │    │ read_page (a11y),    │                 │
│  │  q=title&loc=    │    │ NOT JavaScript       │                 │
│  └─────────────────┘    └─────────────────────┘                  │
│                                                                   │
│  Channel Selection Logic:                                         │
│    if (browser_available) → Ch1 + Ch2 + Ch3 + Ch4               │
│    if (!browser_available) → Ch1 + Ch3 only                      │
│    Sunday/holiday → expect Ch4 low yield                         │
└──────────────────────────────────────────────────────────────────┘
```

---

## Signal Extraction Pipeline

Raw JD text is converted to structured, typed signals before any scoring occurs:

```
┌──────────────────────────────────────────────────────────────────┐
│                  SIGNAL EXTRACTION (Phase 4.5)                    │
│                                                                   │
│  Input: Raw JD text (up to 3000 chars)                           │
│                                                                   │
│  ┌──────────────────────────────────────────┐                    │
│  │           ROLE SIGNALS                    │                    │
│  │                                           │                    │
│  │  seniority_level:  IC / Manager / Sr Mgr  │                   │
│  │  leadership_scope: team size, reports-to   │                   │
│  │  ic_vs_management: ratio or pure          │                    │
│  │  span_of_influence: org / cross-func      │                    │
│  └──────────────────────────────────────────┘                    │
│                        │                                          │
│  ┌──────────────────────────────────────────┐                    │
│  │          DOMAIN SIGNALS                   │                    │
│  │                                           │                    │
│  │  8 domain types with keyword detection:   │                    │
│  │  • Product Data Science                   │                    │
│  │  • AI/ML Platform                         │                    │
│  │  • Applied AI                             │                    │
│  │  • Experimentation                        │                    │
│  │  • Data Platform Leadership               │                    │
│  │  • Growth Analytics                       │                    │
│  │  • Security/SaaS                          │                    │
│  │  • Strategic Finance                      │                    │
│  └──────────────────────────────────────────┘                    │
│                        │                                          │
│  ┌──────────────────────────────────────────┐                    │
│  │        TECHNOLOGY SIGNALS                 │                    │
│  │                                           │                    │
│  │  Per skill, classified by match:          │                    │
│  │                                           │                    │
│  │  DIRECT       → Python ✓, SQL ✓          │                    │
│  │  TRANSFERABLE → Spark (has Snowflake)     │                    │
│  │  GAP          → Kubernetes (not listed)   │                    │
│  │  LEARNABLE    → Go (has Python base)      │                    │
│  │                                           │                    │
│  │  Categories: Languages, ML/AI, Data       │                    │
│  │  Platforms, Orchestration, Viz, GenAI     │                    │
│  └──────────────────────────────────────────┘                    │
│                        │                                          │
│  ┌──────────────────────────────────────────┐                    │
│  │        COMPANY SIGNALS                    │                    │
│  │                                           │                    │
│  │  growth_stage:    startup / growth / IPO  │                    │
│  │  hiring_velocity: aggressive / steady     │                    │
│  │  team_maturity:   new / established       │                    │
│  │  ats_type:        Greenhouse / Lever / .. │                    │
│  │  hiring_urgency:  urgent / standard       │                    │
│  └──────────────────────────────────────────┘                    │
│                                                                   │
│  Output: Structured signal block (one per role)                  │
│  Used by: Phase 5 scoring, Phase 5.5 network eval               │
│                                                                   │
│  ENFORCEMENT: Phase 5 CANNOT start without signal blocks         │
└──────────────────────────────────────────────────────────────────┘
```

---

## Three-Score Scoring System

```
┌──────────────────────────────────────────────────────────────────┐
│                    SCORING ENGINE (Phase 5)                       │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │                    FIT SCORE (0-100)                      │    │
│  │                                                           │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐                 │    │
│  │  │ Level    │ │ Function │ │ Core     │                  │    │
│  │  │ Match    │ │ Match    │ │ Skills   │                  │    │
│  │  │ (25 pts) │ │ (25 pts) │ │ (25 pts) │                 │    │
│  │  └──────────┘ └──────────┘ └──────────┘                  │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐                 │    │
│  │  │ Domain   │ │ GenAI    │ │ Recency  │                  │    │
│  │  │ Transfer │ │ Bonus    │ │ +Clarity │                  │    │
│  │  │ (10 pts) │ │ (5 pts)  │ │ (10 pts) │                 │    │
│  │  └──────────┘ └──────────┘ └──────────┘                  │    │
│  │                                                           │    │
│  │  Labels: 85+ EXCELLENT │ 70-84 STRONG │ 55-69 MODERATE  │    │
│  └─────────────────────────────────────────────────────────┘     │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │               OPPORTUNITY SCORE (0-100)                   │    │
│  │                                                           │    │
│  │  Culture &    Compensation    Role        Speed-to    ATS │   │
│  │  Growth          25%        Leverage      Hire      Fric │   │
│  │    15%                        20%          20%       20% │   │
│  │  ┌──────┐    ┌──────┐     ┌──────┐    ┌──────┐  ┌─────┐│   │
│  │  │██████│    │██████│     │██████│    │██████│  │█████││   │
│  │  │██████│    │██████│     │██████│    │██████│  │█████││   │
│  │  └──────┘    └──────┘     └──────┘    └──────┘  └─────┘│   │
│  └─────────────────────────────────────────────────────────┘     │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │           INTERVIEW PROBABILITY (0-100)                   │    │
│  │                                                           │    │
│  │  23 signals combined:                                     │    │
│  │                                                           │    │
│  │  POSITIVE                    NEGATIVE                     │    │
│  │  +20  Strong referral        -25  Stale posting (120d+)  │    │
│  │  +15  Direct HM contact      -15  Black hole company     │    │
│  │  +10  Alumni connection      -10  Complex ATS             │    │
│  │  +10  Hiring urgency         -10  Overqualified signals   │    │
│  │  + 5  Recent posting          -5  No warm path            │    │
│  │  + 5  Team growth signals     -5  Saturated applicant pool│    │
│  │  ...                         ...                          │    │
│  │                                                           │    │
│  │  Time decay: 0-7d: +5 │ 8-30d: 0 │ 31-60d: -10         │    │
│  │              61-90d: -15 │ 91-120d: -20 │ 120+d: -25     │    │
│  │                                                           │    │
│  │  Momentum stacking: up to 9 positive signals, cap at 99  │    │
│  └─────────────────────────────────────────────────────────┘     │
│                                                                   │
│  Tie-break: IP → Bucket → Fit → Opportunity                     │
└──────────────────────────────────────────────────────────────────┘
```

---

## Action Gate Decision Engine

The decision engine maps scores to actions with explicit reasoning requirements:

```
┌──────────────────────────────────────────────────────────────────┐
│                   ACTION GATE DECISION TREE                       │
│                                                                   │
│                    ┌─────────────┐                                │
│                    │  Scores In  │                                │
│                    │ Fit/Opp/IP  │                                │
│                    └──────┬──────┘                                │
│                           │                                       │
│                    ┌──────┴──────┐                                │
│                    │ Cooldown    │── YES ──► SKIP (cooldown)      │
│                    │ Active?     │                                │
│                    └──────┬──────┘                                │
│                           │ NO                                    │
│                    ┌──────┴──────┐                                │
│                    │ Black Hole  │── YES ──► SKIP (black hole)   │
│                    │ Company?    │                                │
│                    └──────┬──────┘                                │
│                           │ NO                                    │
│                    ┌──────┴──────┐                                │
│              ┌─YES─┤ Strong Path │                                │
│              │     │ AND IP≥60   │                                │
│              │     │ AND Fit≥70  │                                │
│              │     │ AND Opp≥65  │                                │
│              │     └──────┬──────┘                                │
│              │            │ NO                                    │
│              ▼     ┌──────┴──────┐                                │
│        ┌──────┐    │  IP≥50 AND  │                               │
│        │APPLY │    │  Fit≥65 AND │── YES ──► NETWORK FIRST       │
│        │ NOW  │    │  needs warm │                                │
│        └──────┘    │  intro?     │                                │
│                    └──────┬──────┘                                │
│                           │ NO                                    │
│                    ┌──────┴──────┐                                │
│                    │  IP<40 AND  │                                │
│                    │  Weak path  │── YES ──► SKIP                │
│                    │  AND Fit<70 │                                │
│                    └──────┬──────┘                                │
│                           │ NO                                    │
│                           ▼                                       │
│                    ┌──────────┐                                   │
│                    │   HOLD   │                                   │
│                    │          │                                   │
│                    │ Set gate:│                                   │
│                    │ - timer  │                                   │
│                    │ - trigger│                                   │
│                    │ - action │                                   │
│                    └──────────┘                                   │
│                                                                   │
│  Adaptive Thresholds (if <5 APPLY NOW):                          │
│    Iteration 1: Lower IP to 50, Fit to 65, Opp to 55            │
│    Iteration 2: Check for NETWORK FIRST promotions               │
│    Iteration 3: Expand company list                              │
│    Hard floors: Fit 65, IP 50, Opp 55 (never go below)          │
└──────────────────────────────────────────────────────────────────┘
```

---

## Networking CRM State Machine

The CRM tracks contacts through an 8-state pipeline:

```
┌──────────────────────────────────────────────────────────────────┐
│              NETWORKING PIPELINE STATE MACHINE                    │
│                                                                   │
│  ┌────────────┐     ┌──────────────────┐     ┌──────────────┐   │
│  │ DISCOVERED │────►│ CONTACT_IDENTIFIED│────►│OUTREACH_SENT │   │
│  │            │     │                   │     │              │   │
│  │ Found role,│     │ Found specific    │     │ Message sent │   │
│  │ need to    │     │ person to reach   │     │ waiting for  │   │
│  │ find who   │     │ out to            │     │ response     │   │
│  └────────────┘     └──────────────────┘     └──────┬───────┘   │
│                                                      │           │
│                              ┌───────────────────────┘           │
│                              ▼                                    │
│                     ┌─────────────────┐     ┌────────────────┐  │
│                     │RESPONSE_RECEIVED│────►│REFERRAL_SECURED│  │
│                     │                 │     │                │  │
│                     │ Got a reply     │     │ Person agreed  │  │
│                     │ (positive/neg)  │     │ to refer       │  │
│                     └─────────────────┘     └───────┬────────┘  │
│                                                     │            │
│                              ┌──────────────────────┘            │
│                              ▼                                    │
│                  ┌────────────────────┐    ┌─────────────────┐  │
│                  │APPLICATION_SUBMITTED│───►│ INTERVIEW_STAGE │  │
│                  │                    │    │                 │  │
│                  │ Applied with       │    │ Got interview   │  │
│                  │ referral           │    │ (any stage)     │  │
│                  └────────────────────┘    └────────┬────────┘  │
│                                                     │            │
│                                                     ▼            │
│                                             ┌──────────────┐    │
│                                             │    CLOSED     │    │
│                                             │              │    │
│                                             │ hired / pass │    │
│                                             │ / withdrew   │    │
│                                             └──────────────┘    │
│                                                                   │
│  Relationship Strength Transitions:                              │
│    Referral secured  →  Weak ──► Moderate                        │
│    Interview from referral  →  Moderate ──► Strong               │
│    No response after 2 follow-ups  →  (no change, flag stale)   │
│                                                                   │
│  Follow-up Rules:                                                │
│    • 5-7 days after outreach: gentle follow-up                   │
│    • 14+ days no response: final follow-up                       │
│    • After referral secured: weekly status check                 │
└──────────────────────────────────────────────────────────────────┘
```

---

## Interview Pipeline

```
┌──────────────────────────────────────────────────────────────────┐
│                  INTERVIEW STAGE PROGRESSION                      │
│                                                                   │
│  ┌─────────┐   ┌──────────┐   ┌───────────┐   ┌─────────────┐  │
│  │ Applied │──►│  Phone   │──►│ Technical │──►│   Onsite    │  │
│  │         │   │  Screen  │   │  Screen   │   │   Round     │  │
│  └─────────┘   └──────────┘   └───────────┘   └──────┬──────┘  │
│                                                       │          │
│       ┌───────────────────────────────────────────────┘          │
│       ▼                                                          │
│  ┌────────────┐   ┌─────────────┐   ┌────────────────────┐     │
│  │ Team Match │──►│   Hiring    │──►│   Offer Stage      │     │
│  │            │   │  Committee  │   │                    │     │
│  └────────────┘   └─────────────┘   └────────────────────┘     │
│                                                                   │
│  Each stage auto-generates:                                      │
│    ☐ Preparation tasks (company research, STAR stories, etc.)   │
│    ☐ Thank-you note templates                                    │
│    ☐ Timeline expectations                                       │
│    ☐ Follow-up reminders                                         │
└──────────────────────────────────────────────────────────────────┘
```

---

## Learning Loop

The system captures outcomes and adjusts strategy continuously:

```
┌──────────────────────────────────────────────────────────────────┐
│                     LEARNING FEEDBACK LOOP                        │
│                                                                   │
│  ┌──────────────┐                                                │
│  │ Run Outcomes  │    (interviews, rejections, ghosted)          │
│  └──────┬───────┘                                                │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────────────────────────────────────────────┐        │
│  │              OUTCOME PROCESSING                       │       │
│  │                                                       │       │
│  │  ┌─────────────────┐  ┌─────────────────────────┐   │       │
│  │  │ Channel Metrics  │  │ Cluster Metrics          │  │       │
│  │  │                  │  │                          │  │       │
│  │  │ Per channel:     │  │ Per cluster:             │  │       │
│  │  │  • # applied     │  │  • interview rate        │  │       │
│  │  │  • # responded   │  │  • avg time to response  │  │       │
│  │  │  • # interviewed │  │  • conversion patterns   │  │       │
│  │  │  • # offered     │  │                          │  │       │
│  │  │  • conversion %  │  │ 8 clusters:              │  │       │
│  │  └─────────┬────────┘  │  Product DS, AI/ML,      │  │       │
│  │            │           │  Growth, Experimentation, │  │       │
│  │            │           │  Data Platform, Applied,  │  │       │
│  │            │           │  Security/SaaS, Strategic │  │       │
│  │            │           └──────────┬───────────────┘  │       │
│  │            │                      │                   │       │
│  └────────────┼──────────────────────┼───────────────────┘       │
│               │                      │                           │
│               ▼                      ▼                           │
│  ┌─────────────────────────────────────────────────────┐        │
│  │              STRATEGY ADJUSTMENTS                     │       │
│  │                                                       │       │
│  │  Channel adjustments:                                 │       │
│  │    High conversion → allocate more search time        │       │
│  │    Low conversion  → reduce or skip channel           │       │
│  │                                                       │       │
│  │  Score adjustments:                                   │       │
│  │    Cluster interview_rate > 30% → +5 IP boost         │       │
│  │    Cluster interview_rate < 10% → -5 IP penalty       │       │
│  │                                                       │       │
│  │  Company patterns:                                    │       │
│  │    3+ apps, 0 interviews → "cold" flag                │       │
│  │    5+ apps, 0 interviews → "black hole" (auto-SKIP)   │      │
│  │                                                       │       │
│  │  Keyword refinement:                                  │       │
│  │    Track which keywords produce scored roles           │       │
│  │    Drop keywords with 0 results over 3+ runs          │       │
│  └─────────────────────────────────────────────────────┘        │
│               │                                                  │
│               ▼                                                  │
│        Next run uses updated weights                             │
└──────────────────────────────────────────────────────────────────┘
```

---

## State Management

All state lives in a single JSON preferences file:

```
preferences.json
│
├── candidate_profile          # Static: skills, experience, targets
│   ├── technical_skills
│   ├── domain_expertise
│   ├── semantic_skill_clusters
│   └── transferable_patterns
│
├── search_config              # Semi-static: search parameters
│   ├── keywords[]
│   ├── target_companies[]
│   └── location_preferences
│
├── company_search_log         # Dynamic: per-company search history
│   └── {company}: { times_searched, new_roles_found_last }
│
├── company_intelligence       # Dynamic: company health tracking
│   └── {company}: { status, modifier, last_checked, notes }
│
├── keyword_wisdom             # Dynamic: keyword effectiveness
│   └── {keyword}: { times_used, last_used }
│
├── application_cooldowns      # Dynamic: cooling-off periods
│   └── {company}: { applied_date, cooldown_until }
│
├── networking_crm             # Dynamic: all contacts
│   └── contacts[]: { id, name, company, strength, pipeline_state }
│
├── opportunity_pipelines      # Dynamic: all opportunities
│   └── {opp_id}: { stage, company, role, scores, dates }
│
├── interview_pipeline         # Dynamic: active interviews
│   └── {interview_id}: { stages[], prep_tasks[], thank_yous[] }
│
├── referral_tracking          # Dynamic: referral search status
│   └── {role}: { status, date, contacts }
│
├── learning_history           # Append-only: strategy log
│   └── strategy_changes[]
│
├── market_health              # Dynamic: market conditions
│   └── { trend, url_404_rate, dead_channels }
│
├── sourcing_performance       # Dynamic: channel conversion rates
├── channel_performance        # Dynamic: detailed channel metrics
├── cluster_outcomes           # Dynamic: cluster success rates
│
├── run_counter                # Counter: current run number
├── last_run                   # Timestamp: last execution
└── last_sheet_write_status    # Status: last Google Sheet write
```

---

## Data Flow Diagram

End-to-end data flow from search to Google Sheet:

```
Job Boards / LinkedIn / ATS
        │
        ▼
   ┌─────────┐
   │ Raw JD  │    (title, company, location, full text)
   │ Content │
   └────┬────┘
        │
        ▼
   ┌─────────────┐
   │   Signal     │    (role, domain, tech, company signals)
   │ Extraction   │
   └──────┬──────┘
          │
          ▼
   ┌──────────────┐
   │   Scoring     │    (Fit, Opportunity, IP scores)
   │   Engine      │
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐
   │  Action Gate  │    (APPLY NOW / NETWORK FIRST / HOLD / SKIP)
   │  + Network   │
   │    Eval       │
   └──────┬───────┘
          │
    ┌─────┼──────────────────┐
    ▼     ▼                  ▼
┌──────┐ ┌────────────┐ ┌──────────┐
│Action│ │   26-col   │ │ Summary  │
│ Pack │ │    CSV     │ │  Report  │
│(.md) │ │  (.csv)    │ │ (inline) │
└──────┘ └─────┬──────┘ └──────────┘
               │
               ▼
        ┌──────────────┐
        │ Google Sheet  │    (via Apps Script injection)
        │ (persistent)  │
        └──────────────┘
               │
               ▼
        ┌──────────────┐
        │ preferences  │    (learning loop closes the cycle)
        │   .json      │
        └──────────────┘
```

---

## Module Dependency Graph

How the 13 files relate to each other:

```
                         SKILL.md
                    (orchestrator)
                    ┌─────┼─────┐
           ┌───────┘     │     └───────┐
           ▼             ▼             ▼
      onboarding    career-sites   linkedin-browser
      (first run)   (Ch1 + Ch3)    (Ch2 + Ch4)
                         │              │
                         └──────┬───────┘
                                ▼
                        signal_extraction
                        (Phase 4.5)
                                │
                                ▼
                            scoring
                          (Phase 5)
                           ┌───┼───┐
                           │   │   │
                           ▼   │   ▼
              networking_  │  decision_engine
              strategy     │  (reasoning reqs)
              (Phase 5.5)  │
                           │
                           ▼
                      guardrails
                    (hard constraints)
                           │
                    ┌──────┼──────┐
                    ▼      ▼      ▼
              output-   networking  interview_
              formats     _crm     pipeline
              (6-9)    (contacts)  (stages)
                           │          │
                           └────┬─────┘
                                ▼
                          learning_loop
                          (Phase 10)
```

---

## Design Rationale

### Why Structured Signals Before Scoring?
Early versions scored directly from JD text. This led to inconsistent results — the same JD would get different scores depending on Claude's "mood" (attention patterns). Extracting typed signals first creates a stable intermediate representation that scoring can reliably operate on.

### Why 23 IP Signals Instead of a Simple Formula?
Interview probability is inherently multi-dimensional. A strong referral at a black-hole company is different from a cold application at a fast-growing startup. The signal-based approach captures these interactions naturally.

### Why a State Machine for CRM?
Networking relationships follow a predictable lifecycle. The 8-state machine prevents skipping steps (you can't go from DISCOVERED to REFERRAL_SECURED without outreach) and enables automated follow-up reminders at appropriate intervals.

### Why Hard Floors on Adaptive Thresholds?
Without floors, the system would eventually recommend applying to everything. The floors (Fit 65, IP 50, Opp 55) represent the minimum where an application is not a waste of time, based on empirical observation across 19 runs.

### Why Apps Script Injection Instead of Google Sheets API?
The skill runs in Claude's sandbox without persistent API credentials. Apps Script injection via the browser is a pragmatic solution — write the function, run it, done. No OAuth flow, no token management.
