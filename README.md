# Job Search AI Skill

**An AI-native job search system that turns Claude into a personal career intelligence agent.**

Built as a [Claude Code skill](https://docs.claude.com), this system transforms job searching from a manual, ad-hoc process into a structured, data-driven pipeline — complete with multi-channel sourcing, automated scoring, relationship CRM, and continuous learning.

> This project started as a personal experiment: *"What if I gave an AI agent the same frameworks, heuristics, and decision criteria that a great recruiter or career coach would use?"* Over 19 iterations, it evolved into a comprehensive system that manages the full lifecycle of a job search.

---

## Why This Exists

Traditional job searching is broken:

- You spend hours scrolling job boards, reading the same boilerplate JDs
- You apply to everything, hear back from almost nothing
- You lose track of who you talked to, which roles you applied for, and what follow-ups are due
- You have no systematic way to learn from what works and what doesn't

This skill addresses all of that by giving Claude a structured framework to operate as your personal job search agent — discovering opportunities, evaluating fit, managing relationships, and learning from outcomes over time.

**The core insight**: Warm channels (referrals, networking) convert at 80-100%, while cold applications convert at 0-5%. The entire system is designed around this reality.

---

## Architecture Overview

> For the full deep-dive with all diagrams, see **[ARCHITECTURE.md](ARCHITECTURE.md)**.

```
┌─────────────────────────────────────────────────────────────┐
│                    EXECUTION PIPELINE                        │
│                                                              │
│  Phase 0   Load Preferences & State                         │
│  Phase 1   Daily Check-in (CRM follow-ups, interviews)      │
│  Phase 2   Search Strategy (keywords, company intel)         │
│  Phase 3   Multi-Channel Search (4 channels)                │
│  Phase 4   Read & Extract JD Signals                        │
│  Phase 5   Score & Classify (Fit/Opportunity/IP)            │
│  Phase 6   Generate Action Pack                             │
│  Phase 7   Export CSV                                       │
│  Phase 8   Summary Report                                   │
│  Phase 9   Write to Google Sheets                           │
│  Phase 10  Save Preferences & Learning                      │
└─────────────────────────────────────────────────────────────┘

┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐
│  Opportunity  │  │ Relationship │  │  Interview Pipeline  │
│  Discovery    │  │     CRM      │  │     Management       │
│              │  │              │  │                      │
│ 4 channels   │  │ 8-state      │  │ Stage tracking       │
│ Signal       │  │ pipeline     │  │ Prep tasks           │
│ extraction   │  │ Follow-up    │  │ Thank-you notes      │
│ 3-score      │  │ automation   │  │ Outcome tracking     │
│ system       │  │ Strength     │  │                      │
│ Action gates │  │ transitions  │  │                      │
└──────────────┘  └──────────────┘  └──────────────────────┘
         │                │                    │
         └────────────────┼────────────────────┘
                          │
              ┌───────────┴───────────┐
              │    Learning Loop      │
              │                       │
              │ Channel performance   │
              │ Cluster outcomes      │
              │ Score adjustments     │
              │ Strategy evolution    │
              └───────────────────────┘
```

### Three Integrated Domains

**1. Opportunity Discovery** — The search engine. Runs 4 parallel channels to find roles:

| Channel | Method | Best For |
|---------|--------|----------|
| Career Sites | Direct company site scraping | Targeted companies |
| LinkedIn Jobs | Browser automation + JS extraction | Broad discovery |
| Boolean/ATS | Greenhouse, Lever, Ashby API patterns | Hidden roles |
| LinkedIn Posts | Recruiter/HM post mining | Warm leads |

**2. Relationship CRM** — Tracks every contact through an 8-state pipeline:

```
DISCOVERED → CONTACT_IDENTIFIED → OUTREACH_SENT → RESPONSE_RECEIVED
    → REFERRAL_SECURED → APPLICATION_SUBMITTED → INTERVIEW_STAGE → CLOSED
```

**3. Interview Pipeline** — Manages active interviews with stage tracking, auto-generated prep tasks, and thank-you note management.

---

## The Three-Score System

Every role gets scored on three independent dimensions:

### Fit Score (0-100)
*"How well does this role match the candidate?"*

| Dimension | Weight | What It Measures |
|-----------|--------|-----------------|
| Level Match | 25 | Seniority alignment |
| Function Match | 25 | IC vs management, scope |
| Core Skills | 25 | Technical skill overlap |
| Domain Transfer | 10 | Industry relevance |
| GenAI Bonus | 5 | AI/ML component |
| Recency + Clarity | 10 | JD freshness, specificity |

### Opportunity Score (0-100)
*"How good is this opportunity independent of fit?"*

Five pillars: Culture & Growth (15%), Compensation (25%), Role Leverage (20%), Speed-to-Hire (20%), ATS Friction (20%).

### Interview Probability (0-100)
*"What are the realistic chances of getting an interview?"*

This is where the system gets interesting. IP incorporates 23 signals including networking path strength (+20 for strong referral), posting age decay, company hiring velocity, ATS complexity, and accumulated learning from past outcomes.

### Action Gates

Scores combine into four action classifications:

| Gate | Criteria | What Happens |
|------|----------|-------------|
| **APPLY NOW** | Strong path AND IP≥60 AND Fit≥70 AND Opp≥65 | Full application package generated |
| **NETWORK FIRST** | Fit≥65 AND IP≥50 but needs warm intro | Referral search plan created |
| **HOLD** | Promising but missing something | Decision gate + timeline set |
| **SKIP** | IP<40 AND weak network AND Fit<70 | Logged and archived |

---

## Signal Extraction Pipeline

Before scoring, every JD goes through structured signal extraction (Phase 4.5). This prevents the "read the JD and guess" failure mode by converting unstructured text into four categories of typed signals:

```
Raw JD Text
    │
    ▼
┌─────────────────────────────────────┐
│         Signal Extraction           │
│                                     │
│  Role Signals                       │
│    • seniority, scope, IC/mgmt      │
│                                     │
│  Domain Signals                     │
│    • 8 domain types with keywords   │
│                                     │
│  Technology Signals                 │
│    • match strength per skill       │
│    • direct / transferable / gap    │
│                                     │
│  Company Signals                    │
│    • growth stage, hiring urgency   │
│    • ATS type, team maturity        │
└─────────────────────────────────────┘
    │
    ▼
  Scoring (uses signals, not raw text)
```

This separation ensures scoring is consistent, auditable, and doesn't double-count signals.

---

## The Learning Loop

The system gets smarter over time by tracking outcomes across multiple dimensions:

- **Channel Performance**: Which sourcing channels produce interviews? (e.g., "LinkedIn warm intros: 3 interviews from 5 outreach = 60% conversion")
- **Cluster Outcomes**: Which types of roles convert? (e.g., "Growth Analytics roles: 2/8 interview rate → apply -5 IP penalty")
- **Hot/Cold Patterns**: Which companies respond vs ghost? Track as "hot" or "black hole"
- **Score Calibration**: If high-scored roles don't convert, adjust weights. If low-scored roles surprise, investigate why.
- **Adaptive Thresholds**: When too few roles pass APPLY NOW gates, the system automatically lowers thresholds (with hard floors) rather than producing empty results.

---

## Operational Modes

### Discovery Mode (Weekly)
Full 10-phase pipeline. Searches all channels, scores new roles, generates action packs, updates the Google Sheet.

### Networking Mode (Daily)
Lightweight. Checks CRM follow-ups, interview pipeline updates, and manages outreach — no new searching.

---

## Guardrails

The system includes hard constraints to prevent counterproductive behavior:

- **No spray-and-pray**: Max 3 applications/day, minimum Fit 65
- **No recruiter spam**: Max 2 outreach messages/week per company
- **Auto-SKIP low probability**: IP<40 AND weak network AND Fit<70
- **Respect cooldowns**: If you recently applied to a company, enforce the waiting period
- **Warm-first always**: If a warm channel exists, use it (80-100% vs 0-5%)
- **Black hole detection**: 5+ applications with 0 interviews → company gets flagged

---

## File Structure

```
skill/
├── SKILL.md                          # Main execution guide (10-phase pipeline)
└── references/
    ├── career-sites.md               # Company-specific navigation patterns
    ├── decision_engine.md            # Reasoning requirements for all decisions
    ├── guardrails.md                 # Hard constraints and anti-patterns
    ├── interview_pipeline.md         # Interview stage tracking & prep tasks
    ├── learning_loop.md              # Continuous learning system
    ├── linkedin-browser.md           # LinkedIn browser automation (Ch 2 & 4)
    ├── networking_crm.md             # Contact tracking & 8-state pipeline
    ├── networking_strategy.md        # Path strength classification
    ├── onboarding.md                 # First-run setup flow
    ├── output-formats.md             # CSV schema, Action Pack, Google Sheets
    ├── scoring.md                    # Three-score system & action gates
    └── signal_extraction.md          # JD → structured signals pipeline
```

---

## How to Use

### Prerequisites
- [Claude Desktop](https://claude.ai) with Cowork mode or [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- A Google Sheet for tracking (optional but recommended)
- LinkedIn account for browser-based channels

### Installation

1. Copy the `skill/` directory into your Claude skills folder:
   ```
   ~/.skills/skills/job-search/
   ```
   Or wherever your skill directory is configured.

2. The skill triggers automatically when you mention job search-related keywords like "run my job search", "find roles matching my resume", "networking update", etc.

3. On first run, the onboarding flow will guide you through setting up your candidate profile, target companies, and preferences.

### Quick Start

Just tell Claude:
```
"Run my job search"
```

The system will walk you through onboarding (first time) or execute the full discovery pipeline (subsequent runs).

---

## Design Decisions & Lessons Learned

### Why a Skill, Not a Traditional App?

A skill is essentially a prompt — a structured set of instructions that gives Claude domain expertise. This approach has several advantages over building a traditional application:

1. **No infrastructure**: No servers, databases, or deployment pipelines. The "backend" is a JSON preferences file.
2. **Natural language interface**: Users interact conversationally, not through forms and buttons.
3. **Adaptive execution**: Claude can adjust the pipeline based on context (Sunday night → expect fewer recruiter posts, same-day re-run → adjust expectations).
4. **Browser integration**: Through Cowork mode, the skill can directly browse LinkedIn, read JDs, and interact with Google Sheets.

### Why 13 Files Instead of 1?

Early versions were a single massive prompt. This failed because:
- Claude's context window would fill up with instructions before doing any actual work
- Changes to one component (e.g., scoring) required re-reading the entire system
- Reference files load on-demand — the scoring rubric is only loaded during Phase 5

### The Warm-First Philosophy

The most impactful design decision was making networking the primary channel, not an afterthought. The data is clear: cold applications at senior levels have near-zero conversion rates. The entire scoring system, action gates, and CRM are oriented around this reality.

### Adaptive Thresholds

A common failure mode: the scoring system is too strict and produces 0 APPLY NOW results. Rather than forcing users to manually adjust thresholds, the system automatically relaxes criteria (with hard floors) when the pipeline runs dry. This prevents the "perfect is the enemy of good" trap.

---

## What I Learned Building This

This project was an exploration of what happens when you treat an AI agent as a collaborative partner rather than a tool. Some observations:

1. **Structure matters more than intelligence**. Claude is already smart enough to evaluate job descriptions. What it needs is a framework — scoring rubrics, decision trees, state machines — to apply that intelligence consistently across 50+ roles per session.

2. **State management is the hard problem**. The preferences JSON file is essentially a hand-rolled database. Tracking contacts, cooldowns, interview stages, and learning history across sessions required careful schema design.

3. **Prompt engineering is software engineering**. This project has version control, modular architecture, defined interfaces between components, and test-equivalent verification steps. The only difference is the "code" is natural language.

4. **The 80/20 of job searching is relationships**. Building the CRM and networking strategy components had more impact on actual job search outcomes than optimizing the scoring algorithm.

5. **AI agents need guardrails, not just goals**. Without explicit constraints (max applications/day, minimum fit scores, cooldown enforcement), the system would optimize for volume over quality.

---

## Evolution

This skill has gone through 15 major versions and 19 execution runs:

- **v1-3**: Basic job scraping + simple scoring
- **v4-6**: Added multi-channel search, structured signal extraction
- **v7-9**: Introduced the three-score system, action gates
- **v10-12**: Built the networking CRM, relationship strength tracking
- **v13-14**: Added interview pipeline, learning loop, browser automation
- **v15**: Current version — full integration of all three domains with adaptive thresholds

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

## Author

Built by exploring the intersection of AI agents and real-world workflow automation. This is an open-source showcase of what's possible when you give an AI structured domain expertise and let it operate as a collaborative partner.

If you find this useful or have ideas for improvement, feel free to open an issue or PR.
