# Job Search AI Skill

**Most Claude skills are a single file. This one is 17 modules.**

An AI-native job search system that turns Claude into a personal career intelligence agent — with multi-channel sourcing, three-score evaluation, a relationship CRM, and continuous learning. All markdown, no code. No Python, no APIs, no database. Just structured prose that runs anywhere Claude runs.

> This project started as a personal experiment: *"What if I gave an AI agent the same frameworks, heuristics, and decision criteria that a great recruiter or career coach would use?"* Over 19+ iterations, it evolved into a comprehensive system that manages the full lifecycle of a job search — built entirely in natural language.

<!-- 🎬 [Watch the demo video](TODO: add link) -->

---

## What is a Claude Code Skill?

A **skill** is a structured prompt — a set of natural language instructions that gives Claude deep domain expertise in a specific area. Think of it as "programming with prose."

Unlike traditional software:
- **No servers or databases** — state lives in a JSON file and a Google Sheet
- **No deployment pipeline** — copy a folder, start talking to Claude
- **No UI to build** — the interface is natural language conversation
- **No API integrations to maintain** — Claude browses the web directly via Cowork mode

A skill is activated when Claude detects relevant keywords in conversation (e.g., "run my job search"). It loads the main instruction file (`SKILL.md`), which orchestrates **17 reference modules** on-demand — only pulling in the scoring rubric when scoring, the CRM spec when managing contacts, etc.

**This project demonstrates that prompt engineering at scale is software engineering** — with version control, modular architecture, defined interfaces, and iterative debugging.

---

## Portability: The Agent Skills Standard

This skill follows the emerging **Agent Skills standard** — a folder of markdown files with a `SKILL.md` orchestrator and `references/` directory. Because it's all markdown with no runtime dependencies, it works across:

- **Claude Code** — native skill loading
- **Claude Cowork** — browser-integrated execution
- **Cursor** — as context files
- **Gemini CLI** — as instruction files
- **Codex CLI** — as prompt context

No vendor lock-in. No proprietary format. Copy the folder, point your agent at it, and it works.

---

## 10-Phase Pipeline

Every execution follows a structured 10-phase pipeline:

| Phase | Name | What Happens |
|-------|------|-------------|
| 0.5 | Company Targeting | Dynamic tier rotation & auto-discovery |
| 2B | Keyword Engine | Proven / Promising / Discovery tier management |
| 3 | 4-Channel Search | Career sites, LinkedIn, ATS APIs, recruiter posts |
| 3.5–3.7 | URL Triage | Dead link detection before context investment |
| 4.5 | Signal Extraction | Unstructured JD → structured role/domain/tech signals |
| 5 | Three-Score Engine | Fit + Opportunity + Probability scoring |
| 5.5 | Action Gates | Deterministic routing: APPLY / NETWORK / HOLD / SKIP |
| 6–9 | Action Packs | Auto-generated tailored deliverables + CRM sync |
| 9.5 | CRM Sync | Pipeline state transitions + follow-up scheduling |
| 10 | Learning Loop | Outcome feedback → scoring recalibration |

---

## 5 Domain Clusters — 17 Modules

The 17 reference modules are organized into 5 domain clusters:

### Search
The discovery engine — how the system finds opportunities across multiple channels.
- `channel_search.md` — Multi-channel search orchestrator (Phase 3)
- `career-sites.md` — Company-specific navigation patterns & ATS handling
- `linkedin-browser.md` — LinkedIn browser automation (Channels 2 & 4)
- `keyword_engine.md` — Keyword intelligence with Proven/Promising/Discovery tiers

### Intelligence
The evaluation brain — how the system scores and decides.
- `signal_extraction.md` — JD → structured signals pipeline (anti-hallucination layer)
- `scoring.md` — Three-score system (Fit + Opportunity + Interview Probability)
- `decision_engine.md` — Reasoning requirements for all action decisions

### CRM
Relationship management — the warm-first philosophy in action.
- `networking_crm.md` — Contact tracking & 8-state pipeline
- `networking_strategy.md` — Path strength classification & outreach tactics
- `interview_pipeline.md` — Interview stage tracking & prep task generation

### System
Operational infrastructure — constraints, learning, and output.
- `guardrails.md` — Hard constraints & anti-patterns (8 rules)
- `learning_loop.md` — Continuous learning system & adaptive thresholds
- `output-formats.md` — CSV schema, Action Packs, Google Sheets injection
- `onboarding.md` — First-run setup flow

### Targeting
Pre-search intelligence — who to search and where.
- `company_targeting.md` — Dynamic company queue with cadence tiers
- `url_triage.md` — URL pre-triage & liveness checking (Phases 3.5–3.7)
- `linkedin-company-filters.md` — LinkedIn company ID registry

---

## AI Patterns & Techniques

This project is a case study in building reliable AI agent systems. Here are the transferable patterns, each battle-tested across 19+ execution runs:

### 1. Lazy-Loading References (Context Window Management)
```
                    ┌─────────────┐
                    │   SKILL.md  │  ← Always loaded (~2K tokens)
                    │ Orchestrator│
                    └──────┬──────┘
                           │
              ┌────────────┼────────────────┐
              │            │                │
              ▼            ▼                ▼
         ┌────────┐  ┌──────────┐  ┌────────────┐
         │scoring │  │  crm.md  │  │ guardrails │  ← Loaded on-demand
         │  .md   │  │          │  │    .md     │     (~1K tokens each)
         └────────┘  └──────────┘  └────────────┘

    Problem: A single 30K-token prompt fills the context window
             before the agent does any actual work.
    Solution: Main file is a lightweight orchestrator. 17 reference
              files load only when their phase executes.
    Result: ~80% context window available for actual task execution.
```

### 2. Signal Extraction as Anti-Hallucination Layer
```
    ❌ WITHOUT SIGNALS          ✅ WITH SIGNALS

    Raw JD ──► Score            Raw JD ──► Extract ──► Score
                                           Signals

    "Read this and                "Extract these      "Score these
     give me a                     specific fields     typed fields
     number"                       into this schema"   using this rubric"

    Same JD → different         Same JD → same signals → same score
    scores each time            every time (deterministic intermediate)
```
The key insight: LLMs are inconsistent when asked to go from unstructured text to a number in one step. Adding a structured intermediate representation (typed signals) makes scoring reproducible.

### 3. State Machines in Natural Language
```
    DISCOVERED → CONTACT_IDENTIFIED → OUTREACH_SENT → RESPONSE_RECEIVED
        → REFERRAL_SECURED → APPLICATION_SUBMITTED → INTERVIEW_STAGE → CLOSED

    Enforced in prose: "NEVER skip states. A contact cannot move to
    REFERRAL_SECURED without passing through OUTREACH_SENT first."
```
Natural language state machines give AI agents the same lifecycle guarantees as code-based ones — without requiring a runtime. The prompt *is* the runtime.

### 4. Adaptive Thresholds with Hard Floors
```
    Pipeline produces 0 results?

    Iteration 1: Lower IP 60→50, Fit 70→65, Opp 65→55
    Iteration 2: Promote NETWORK FIRST → APPLY NOW candidates
    Iteration 3: Expand company list
    ─────────────────────────────────────────────────
    HARD FLOOR: Fit ≥ 65, IP ≥ 50, Opp ≥ 55 (NEVER below)

    Without floors → agent eventually recommends everything
    Without adaptation → agent produces empty results on tough days
```

### 5. Guardrails for Agent Autonomy
```
    ┌─────────────────────────────────────────────────┐
    │  HARD CONSTRAINTS (never violated)              │
    │                                                  │
    │  • Max 3 applications/day                       │
    │  • Max 2 outreach messages/week per company     │
    │  • Minimum Fit score 65 to apply                │
    │  • Cooldown enforcement between applications    │
    │  • Black hole detection (5+ apps, 0 interviews) │
    │  • Protect application budgets at key companies  │
    └─────────────────────────────────────────────────┘

    Without guardrails, the agent optimizes for volume.
    With guardrails, it optimizes for quality within bounds.
```

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

IP incorporates 20+ signals including networking path strength (+20 for strong referral), posting age decay, company hiring velocity, ATS complexity, and accumulated learning from past outcomes.

### Action Gates

Scores combine into four action classifications:

| Gate | Criteria | What Happens |
|------|----------|-------------|
| **APPLY NOW** | Strong path AND IP>=60 AND Fit>=70 AND Opp>=65 | Full application package generated |
| **NETWORK FIRST** | Fit>=65 AND IP>=50 but needs warm intro | Referral search plan created |
| **HOLD** | Promising but missing something | Decision gate + timeline set |
| **SKIP** | IP<40 AND weak network AND Fit<70 | Logged and archived |

---

## What I Learned Building This

1. **Structure matters more than intelligence.** Claude is already smart enough to evaluate job descriptions. What it needs is a framework — scoring rubrics, decision trees, state machines — to apply that intelligence consistently across 50+ roles per session.

2. **State management is the hard problem.** Tracking contacts, cooldowns, interview stages, and learning history across sessions required careful schema design — all in a single JSON file.

3. **Prompt engineering is software engineering.** This project has version control, modular architecture, defined interfaces between components, and test-equivalent verification steps. The only difference is the "code" is natural language.

4. **The 80/20 of job searching is relationships.** Building the CRM and networking strategy components had more impact on actual job search outcomes than optimizing the scoring algorithm.

5. **AI agents need guardrails, not just goals.** Without explicit constraints, the system would optimize for volume over quality. Every guardrail exists because the agent *actually did* the thing it prevents.

---

## Architecture Overview

> For the full deep-dive with all diagrams, see **[ARCHITECTURE.md](ARCHITECTURE.md)**.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       JOB SEARCH AI SKILL v16.11                            │
│                                                                             │
│   ┌───────────┐     ┌──────────────┐     ┌──────────────┐                  │
│   │  DISCOVER │────►│   EVALUATE   │────►│     ACT      │                  │
│   │           │     │              │     │              │                  │
│   │ 4-channel │     │ Signal       │     │ APPLY NOW    │                  │
│   │ search    │     │ extraction → │     │ NETWORK FIRST│                  │
│   │           │     │ 3-score      │     │ HOLD         │                  │
│   │ Career    │     │ system →     │     │ SKIP         │                  │
│   │ LinkedIn  │     │ Action gates │     │              │                  │
│   │ ATS/Bool  │     │              │     │ Action packs │                  │
│   │ Recruiter │     │              │     │ CSV + Sheets │                  │
│   │ Posts     │     │              │     │ CRM updates  │                  │
│   └───────────┘     └──────────────┘     └──────┬───────┘                  │
│         ▲                                        │                          │
│         │              ┌─────────────┐           │                          │
│         └──────────────│   LEARN     │◄──────────┘                          │
│                        │             │                                      │
│                        │ Channel     │                                      │
│                        │ performance │                                      │
│                        │ Cluster     │                                      │
│                        │ outcomes    │                                      │
│                        │ Score       │                                      │
│                        │ calibration │                                      │
│                        │ Hot/cold    │                                      │
│                        │ tracking    │                                      │
│                        └─────────────┘                                      │
│                                                                             │
│   ┌──────────────────────────────────────────────────────────┐              │
│   │  RELATIONSHIP CRM                    INTERVIEW PIPELINE  │              │
│   │  8-state pipeline                    Stage tracking       │              │
│   │  Follow-up automation                Prep task generation │              │
│   │  Strength transitions                Thank-you management │              │
│   └──────────────────────────────────────────────────────────┘              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Evolution: What Broke & What I Fixed

This skill has gone through 16 major versions and 19+ execution runs. Here's the real story:

| Version | What Changed | What Broke | How I Fixed It |
|---------|-------------|------------|----------------|
| **v1-3** | Basic job scraping + simple scoring | Scores were random — same JD got different scores each run | Signal extraction: force structured intermediate before scoring |
| **v4-6** | Multi-channel search, signal extraction | Single massive prompt (~30K tokens) filled context window | Modular architecture: split into reference files with lazy-loading |
| **v7-9** | Three-score system, action gates | Too strict — 0 APPLY NOW results on most runs | Adaptive thresholds with hard floors |
| **v10-12** | Networking CRM, relationship tracking | Contacts "jumped" states in the CRM pipeline | Enforced state machine with explicit transition rules in prose |
| **v13-14** | Interview pipeline, browser automation | LinkedIn scraping broke on DOM changes | Switched to accessibility tree parsing with fallback chains |
| **v15** | Full integration, adaptive learning | Channel 4 had zero yield on weekends; learning loop overreacted | Time-aware expectations + dampened learning rate (3+ data points) |
| **v16.x** | Company targeting, keyword engine, URL triage, LinkedIn company filters | Pre-search phases were manual and inconsistent; dead URLs wasted context | Added 5 new modules: automated company tiering, keyword intelligence, URL pre-triage, company ID registry, multi-channel search orchestrator |

### Recurring Themes

- **The biggest improvements came from adding structure, not intelligence.** Claude was always smart enough. The failures were in consistency, not capability.
- **Every "AI problem" was actually a systems design problem.** Hallucinated scores? Add structured signals. Context overflow? Add lazy-loading. Inconsistent state? Add a state machine.
- **Guardrails are features, not limitations.** Every constraint was added because the agent *actually did* the thing it prevents.

---

## Operational Modes

### Discovery Mode (Weekly)
Full 10-phase pipeline. Searches all channels, scores new roles, generates action packs, updates the Google Sheet.

### Networking Mode (Daily)
Lightweight. Checks CRM follow-ups, interview pipeline updates, and manages outreach — no new searching.

---

## File Structure

```
skill/
├── SKILL.md                          # Main orchestrator (10-phase pipeline)
└── references/                       # 17 domain-specific modules
    │
    │  # Search cluster
    ├── channel_search.md             # Multi-channel search orchestrator
    ├── career-sites.md               # Company-specific navigation & ATS patterns
    ├── linkedin-browser.md           # LinkedIn browser automation (Ch 2 & 4)
    ├── keyword_engine.md             # Keyword intelligence engine (Phase 2B)
    │
    │  # Intelligence cluster
    ├── signal_extraction.md          # JD → structured signals pipeline
    ├── scoring.md                    # Three-score system & action gates
    ├── decision_engine.md            # Reasoning requirements for decisions
    │
    │  # CRM cluster
    ├── networking_crm.md             # Contact tracking & 8-state pipeline
    ├── networking_strategy.md        # Path strength classification
    ├── interview_pipeline.md         # Interview stage tracking & prep tasks
    │
    │  # System cluster
    ├── guardrails.md                 # Hard constraints & anti-patterns
    ├── learning_loop.md              # Continuous learning system
    ├── output-formats.md             # CSV schema, Action Packs, Google Sheets
    ├── onboarding.md                 # First-run setup flow
    │
    │  # Targeting cluster
    ├── company_targeting.md          # Dynamic company queue & cadence tiers
    ├── url_triage.md                 # URL pre-triage & liveness (Phases 3.5-3.7)
    └── linkedin-company-filters.md   # LinkedIn company ID registry
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

## Limitations & Known Failure Modes

### Structural Limitations
- **Single-user design**: Scoring weights and domain clusters are tuned for one person's job search. Adapting for a different role/seniority requires reconfiguration.
- **Browser dependency**: Channels 2 and 4 (LinkedIn) require Cowork mode with browser access. Without it, you lose ~50% of sourcing coverage.

### AI-Specific Failure Modes
- **Signal extraction quality depends on JD quality**: Vague JDs produce weak signals and unreliable scores.
- **Learning loop cold start**: First 3-5 runs have no historical data — adaptive adjustments don't kick in until then.
- **Context window pressure on large runs**: Processing 40+ roles in a single session can push against limits even with lazy-loading.

### Operational Gotchas
- **LinkedIn DOM changes**: Browser-based channels break when LinkedIn updates their UI.
- **Google Sheets write failures**: Apps Script injection approach is fragile — browser state or permissions can cause silent failures.

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

## Author

Built by exploring the intersection of AI agents and real-world workflow automation. This is an open-source showcase of what's possible when you give an AI structured domain expertise and let it operate as a collaborative partner.

If you find this useful or have ideas for improvement, feel free to open an issue or PR.
