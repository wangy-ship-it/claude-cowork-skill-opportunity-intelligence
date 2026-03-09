# Onboarding Flow (First Run Only)

This flow runs when `run_counter` is 0 or the preferences file doesn't exist. On subsequent runs, the Daily Check-in replaces this entirely.

## Checkpoint 1: Resume & Minimum Salary

Ask for resume (file path, upload, or pasted text). Confirm extracted profile — role level, key skills, target locations. Also ask minimum base salary (default $185K for SM in high-cost areas).

**Do not proceed until user confirms profile AND salary.**

## Checkpoint 2: Location Preferences

Ask where they want to work (specific cities, regions, remote-friendly, anywhere US). Store and use to filter results.

**Do not proceed until confirmed.**

## Checkpoint 3: Company Targeting

Two modes:
- **Mode A (named companies)**: User provides list. Confirm back.
- **Mode B (discovery)**: User describes what they want. Web search, present 8-15 with rationale.

Classify into buckets:
- **Bucket A**: High-growth startups, Series C+, urgent hiring
- **Bucket B**: Big tech / prestige, slower cycles
- **Bucket C**: Adjacent perfect fit, high conversion

**Do not search until user approves company list.**

## Checkpoint 4: Google Sheet

Ask for Google Sheet URL or CSV save path.

## Quick-Start Shortcut

If all inputs provided in one message, confirm in single message and wait for approval.

## Phase 1: Deep Candidate Analysis

Read resume thoroughly. Extract:
1. **Role level and trajectory** → store as `trajectory_type` (leadership / domain_deepening / broadening / pivot)
2. **Technical depth** → Expert / Proficient / Familiar
3. **Domain expertise** → map to team types
4. **Leadership and influence** → team sizes, budget, stakeholder level
5. **GenAI / emerging tech** → LLMs, RAG, agents
6. **Quantified impact** → dollar amounts, percentages, scale
7. **Education and certifications**

Build a "superpower statement" — 2-3 sentence unique value pitch.

### Enhanced Contextual Matching

1. **Career trajectory matching**: Read JDs through "where going," not just "where been." Leadership trajectory = +1 level stretch is moderate risk.
2. **Semantic skill matching**: "Built churn prediction" ≈ "customer lifetime value modeling." Don't penalize vocabulary differences.
3. **Transferable value detection**: Pattern match across different surface descriptions.
4. **Hidden strength surfacing**: Read team descriptions, "nice to have," company context.
5. **Dynamic JD interpretation**: Decode jargon, infer needs from emphasis, read between lines on level.

### Save Enriched Profile

```json
"candidate_profile": {
  "trajectory_type": "leadership",
  "superpower_statement": "Senior analytics leader who combines...",
  "transferable_patterns": ["predictive_analytics_for_business_impact", ...],
  "semantic_skill_clusters": {
    "ml_and_prediction": ["churn modeling", "A/B testing", ...],
    "leadership": ["team management", "cross-functional", ...],
    "data_infrastructure": ["Snowflake", "Airflow", ...],
    "genai": ["RAG", "LLM evaluation", ...]
  }
}
```
