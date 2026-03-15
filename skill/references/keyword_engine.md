## PHASE 2B: Keyword Intelligence Engine ← BLOCKING GATE

Phase 3 CANNOT start without the keyword matrix from this phase.

1. Load `keyword_wisdom`. Compute confidence: `avg_score × log(times_used + 1)`.
2. Rank: Proven (conf ≥ 50, used ≥ 3) → Promising (30-49 or used < 3) → Underperformers (conf < 30, used ≥ 3).
3. Discovery budget: 1-2 NEW keywords from resume, hot patterns, interview JD language, adjacent titles.
4. Sample-before-commit (FULL_ANALYSIS only): 3 keywords at 2 companies, score 6 results. Swap if avg < 50.

### Company-Specific Title Vocabulary ← NEW in v16.7

Different companies use different naming conventions for similar roles. For example, DoorDash titles most of its data roles as "Analytics" rather than "Data Science" — so searching "data science" on their career site returns near-zero results while "analytics" returns 13+. This vocabulary mismatch is the #1 cause of false-negative pipeline misses at proprietary career sites.

**How it works**: Each company can have a `title_vocabulary` in `company_intelligence` that maps which keywords actually work on their career site:

```json
"company_intelligence": {
  "DoorDash": {
    "title_vocabulary": {
      "high_yield": ["analytics", "insights", "strategy"],
      "low_yield": ["data science"],
      "learned_from": "run12_diagnostic",
      "last_updated": "3/13/2026"
    }
  }
}
```

**During Phase 2B, build a per-company keyword plan:**
1. Check `company_intelligence.{company}.title_vocabulary` for each company in the search queue
2. If `title_vocabulary` exists: use `high_yield` terms as primary keywords for that company's Ch1 search. Demote `low_yield` terms (try them last, or skip if context-constrained)
3. If no `title_vocabulary` exists: use the default keyword matrix. But flag the company for vocabulary discovery in Ch1 (see Phase 3 adaptive keyword expansion)

**Display the company keyword plan alongside the matrix:**
```
COMPANY KEYWORD OVERRIDES (v16.7):
  DoorDash: PRIMARY "analytics", "insights" | DEMOTED "data science" (low_yield)
  Stripe: (no override — using default matrix)
  ...
```

### Global Title Expansion Map ← NEW in v16.9

The market is fragmenting beyond "Data Science Manager." Companies increasingly use adjacent titles for what is functionally the same role. The pipeline needs to catch these systematically, not just when a company-specific vocabulary is learned.

The `title_expansion_map` in preferences defines adjacent title families that the pipeline should always search alongside the primary title. Unlike `title_vocabulary` (which is per-company and learned from career site results), the expansion map is **global** and applies across all channels.

```json
"title_expansion_map": {
  "data_science_manager": {
    "primary": ["data science manager", "senior data science manager"],
    "adjacent": [
      "analytics engineering manager",
      "AI/ML engineering manager",
      "product intelligence lead",
      "decision science manager",
      "applied science manager",
      "machine learning manager"
    ],
    "rationale": "Market title fragmentation — same role, different labels"
  }
}
```

**How it works during Phase 2B:**
1. Load `title_expansion_map` from preferences
2. For each primary title in the keyword matrix, check if adjacent titles exist in the map
3. Add adjacent titles to the **Discovery** tier of the keyword matrix (lower priority than Proven/Promising, but always attempted)
4. Track which adjacent titles produce results — if an adjacent title yields ≥3 scoreable roles across 2+ runs, promote it to Promising

**How it works during Phase 3:**
- **Channel 1 (Career Sites)**: After searching primary keywords, try adjacent titles from the expansion map. This runs BEFORE the adaptive keyword expansion (which handles per-company vocabulary). The sequence is: company vocabulary override → primary matrix → adjacent titles from expansion map → adaptive fallback
- **Channel 2 (LinkedIn)**: Include adjacent titles as additional search queries. LinkedIn's fuzzy matching means these often surface roles the primary title misses
- **Channel 3 (ATS)**: Include adjacent titles in Greenhouse/Ashby/Lever API queries

**Phase 10 integration**: When a run discovers a new title variant that produces high-Fit roles (Fit ≥ 70), propose adding it to the expansion map. Over time the map grows to capture the full vocabulary of the candidate's target role family.

**Display in Phase 2B output:**
```
TITLE EXPANSION MAP (v16.9):
  "data science manager" → also searching: "analytics engineering manager", "AI/ML engineering manager",
    "product intelligence lead", "decision science manager", "applied science manager", "ML manager"
  New this run: (none) | Promotions: (none)
```

### Two Query Types (v16 change)

The keyword matrix includes **two types** of queries that surface fundamentally different role inventories:

- **Title-based queries** — job title searches like "data science manager", "analytics lead". Match on exact role titles.
- **Tool-based queries** — skill/tool searches like "tableau manager analytics", "snowflake experimentation manager data". Surface roles that title-based queries miss (e.g., "Decision Science Lead" requiring Snowflake + dbt won't match "data science manager").

Why both matter: historical data shows tool-based queries produce URLs with **near-100% liveness** compared to title-based at ~15-50%. Tool-based queries find fresher, more specific listings because they match on JD content, not just titles. Both types should be used in Phase 3 — they are complementary.

**Level qualifier rule for tool-based queries**: Tool-based queries that mention only tools without a level indicator (e.g., "snowflake data engineer") will return mostly IC-level roles, which wastes the pipeline on roles below target seniority. Every tool-based query MUST include at least one level qualifier — "manager", "lead", "senior manager", "head of", or "director". Good: `"snowflake" "manager" analytics`. Bad: `"snowflake experimentation data"` (no level → returns junior IC roles). When constructing new tool-based queries, follow the pattern: `{tool} {level} {domain}`.

Load tool-based queries from `search_channels.tool_based_keyword_search.queries` alongside title-based from `search_channels.linkedin_job_board.queries`. Review loaded tool-based queries and add a level qualifier to any that lack one before using them in Phase 3.

**Display the matrix:**
```
KEYWORD MATRIX:
  Title-based:
    Proven: "{kw}" (conf {X}), ...
    Promising: "{kw}" (conf {X}), ...
    Discovery: "{kw}" (new), ...
    Demoted: "{kw}" (conf {X}, skipping)
  Tool-based:
    Proven: "{kw}" (conf {X}), ...
    Promising: "{kw}" (conf {X}), ...
    Discovery: "{kw}" (new), ...
```

After run: update keyword_wisdom (times_used, avg_score, jobs_found, high_score_count, last_used, confidence). Track title-based and tool-based performance **separately** to inform future query budget allocation.

---

