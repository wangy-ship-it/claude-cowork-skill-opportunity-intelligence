# Signal Extraction Layer

This layer runs AFTER reading JDs (Phase 4) and BEFORE scoring (Phase 5). Its purpose is to convert raw job description text into structured signals that scoring can consume deterministically. Scoring should never interpret raw JD text directly — it should only operate on extracted signals.

## Why Signals Before Scoring

Raw JDs are noisy — they mix requirements, nice-to-haves, company boilerplate, and marketing language. Extracting signals first:
1. Makes scoring consistent across different JD writing styles
2. Prevents double-counting (e.g., "ML" appearing 10 times doesn't mean 10x fit)
3. Creates an auditable intermediate representation
4. Enables pattern matching against historical interview-getters

## Signal Categories

### Role Signals

Extract these from each JD:

| Signal | How to Detect | Values |
|--------|--------------|--------|
| **Seniority level** | Title, years required, scope language | IC / Lead / Manager / Sr Manager / Director / VP |
| **Leadership scope** | "manage team of X", "lead org of X", "direct reports" | team_size (integer), org_scope (team / multi-team / org) |
| **IC vs Management** | "hands-on coding" vs "people management", "hiring" | IC / IC-heavy-manager / balanced / management-heavy |
| **Team responsibility** | "build from scratch", "existing team", "grow team" | build_new / grow_existing / maintain / restructure |
| **Span of influence** | Cross-functional mentions, stakeholder level | team_only / cross-functional / executive-facing / C-suite |

### Domain Signals

Map the JD into one or more domain categories:

| Domain | Detection Keywords |
|--------|-------------------|
| **Product Data Science** | product analytics, A/B testing, user behavior, funnel, retention, engagement, product sense |
| **AI/ML Platform** | ML infrastructure, model serving, feature store, ML pipeline, MLOps, training infrastructure |
| **Applied AI** | GenAI, LLM, RAG, agents, NLP, computer vision, recommendation systems |
| **Experimentation** | experimentation platform, A/B testing infrastructure, causal inference, statistical methods |
| **Data Platform Leadership** | data engineering, data warehouse, ETL, pipeline, Snowflake, dbt, Spark, Airflow |
| **Growth Analytics** | growth, acquisition, monetization, pricing, marketplace, LTV, churn |
| **Security/SaaS** | cybersecurity, threat detection, SaaS metrics, enterprise software, B2B analytics |
| **Strategic Finance** | financial analytics, FP&A, revenue forecasting, business intelligence, strategy & ops |

Assign: `primary_domain` (strongest match) and `secondary_domains[]` (weaker matches). If the JD spans multiple domains equally, flag as `multi_domain: true`.

### Technology Signals

Extract required and preferred technologies, then map to the candidate's skill clusters:

| Category | Example Technologies |
|----------|---------------------|
| **Languages** | Python, SQL, R, Scala, Java |
| **ML/AI** | XGBoost, TensorFlow, PyTorch, scikit-learn, LLMs, RAG |
| **Data platforms** | Snowflake, BigQuery, Redshift, Databricks, Spark |
| **Orchestration** | Airflow, dbt, Prefect, Dagster |
| **Experimentation** | Optimizely, internal A/B platforms, Statsig |
| **Visualization** | Tableau, Looker, Mode, Power BI |
| **GenAI stack** | LangChain, vector DBs, prompt engineering, fine-tuning |

For each technology, classify match strength:
- **Direct match**: Candidate has exact skill → `match: direct`
- **Transferable match**: Candidate has equivalent skill → `match: transferable` (e.g., Snowflake ↔ BigQuery)
- **Gap**: Candidate lacks skill and no clear equivalent → `match: gap`
- **Learnable gap**: Candidate lacks but could learn quickly → `match: learnable_gap`

### Company Signals

Extract from the JD and supplement with company_intelligence from preferences:

| Signal | Source | Values |
|--------|--------|--------|
| **Growth stage** | Company intelligence, funding news | pre-seed / seed / series_A-C / growth / public / mega_cap |
| **Hiring expansion** | "growing team", "new team", multiple similar postings | expanding / stable / contracting |
| **Team maturity** | "build from scratch", "established team", "v1 product" | greenfield / early / mature / restructuring |
| **Hiring urgency** | "immediate", "ASAP", "backfill", posting age | urgent / normal / low |
| **ATS type** | URL pattern | greenhouse / lever / workday / ashby / custom |

## Output Format

For each role, produce a structured signal block. Use this exact template — copy it and fill in the values. The reason for enforcing this format is that scoring dimensions map 1:1 to these signal lines, and deviations (like embedding signals in prose or reorganizing them) make scoring less consistent.

```
SIGNALS: {Job Title} @ {Company}
─────────────────────────────────
Role: {seniority} | {ic_vs_mgmt} | team_size: {X} | scope: {span}
Domain: {primary_domain} (+ {secondary_domains})
Tech match: {X} direct, {X} transferable, {X} gaps, {X} learnable
  - Direct: {list}
  - Transferable: {list}
  - Gaps: {list}
  - Learnable: {list}
Company: {growth_stage} | hiring: {expansion} | maturity: {team_maturity}
Urgency: {urgency}
Key requirements: {top 3-5 hard requirements from JD}
Nice-to-haves matched: {count}/{total}
Red flags: {blockers if any — PhD required, wrong location, IC-only, etc.}
Opportunity cluster: {cluster_tag from scoring.md}
```

**Copy-paste example** (filled in):

```
SIGNALS: Sr Manager, Product Data Science @ Gusto
─────────────────────────────────
Role: Sr Manager | balanced | team_size: 6-8 | scope: cross-functional
Domain: product_data_science (+ experimentation, growth_analytics)
Tech match: 5 direct, 2 transferable, 1 gaps, 1 learnable
  - Direct: Python, SQL, A/B testing, cross-functional leadership, product analytics
  - Transferable: Snowflake→BigQuery, Tableau→Looker
  - Gaps: payments domain expertise
  - Learnable: Gusto product context
Company: growth | hiring: expanding | maturity: mature
Urgency: normal
Key requirements: 10+ yrs analytics; 4+ yrs managing DS teams; experimentation design; product sense
Nice-to-haves matched: 3/4
Red flags: none
Opportunity cluster: product_data_science
```

## How Scoring Uses Signals

After extraction, scoring dimensions map directly to signal categories:

| Scoring Dimension | Primary Signals Used |
|-------------------|---------------------|
| Level match (25 pts) | Role signals: seniority, leadership_scope, ic_vs_mgmt |
| Function match (25 pts) | Domain signals: primary_domain, secondary_domains |
| Core skills match (25 pts) | Technology signals: direct/transferable/gap counts |
| Domain transferability (10 pts) | Domain signals + candidate's transferable_patterns |
| GenAI relevance (5 pts) | Technology signals: GenAI stack matches |
| Opportunity Score pillars | Company signals: growth_stage, expansion, maturity |
| Interview Probability | All signals combined + historical match patterns |

## Cluster Tagging

After extracting signals, tag each role with its **opportunity cluster** (see scoring.md § Opportunity Clustering). This enables historical learning about which cluster types produce interviews.

## Enforcement

Signal extraction is a **required step**. Phase 5 (scoring) MUST NOT begin until all scoreable roles have signal blocks produced. The signal block is the input to scoring — if a signal cannot be extracted (e.g., JD too vague), flag the role as `signals_incomplete` and apply a -10 penalty to Interview Probability.
