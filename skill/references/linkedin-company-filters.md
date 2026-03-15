# LinkedIn Company Filters Reference

This reference maps companies to their LinkedIn `f_C` company IDs for targeted job board searches. Use these IDs to search for roles **at a specific company** via LinkedIn's job board, which is the only reliable search surface for companies that use proprietary ATS platforms (no Greenhouse/Ashby/Lever API).

## How It Works

LinkedIn's job search supports a `f_C` URL parameter that filters results to specific companies. Many companies — especially large tech firms — use proprietary ATS systems with no public API. For these companies, the LinkedIn job board with `f_C` filtering is the **primary discovery channel**.

### URL Template

```
https://www.linkedin.com/jobs/search/?keywords={query}&f_C={company_ids}&geoId={geo_id}&f_E={experience_levels}&sortBy=DD
```

### Parameters

| Parameter | Description | Common Values |
|-----------|-------------|---------------|
| `f_C` | Company ID(s), comma-separated (URL-encode commas as `%2C`) | See registry below |
| `geoId` | Geographic filter | `103644278` = United States, `92000000` = Worldwide |
| `f_E` | Experience level | `4` = Mid-Senior, `5` = Director, `4%2C5` = Both |
| `f_TPR` | Time posted | `r3600` = 1hr, `r86400` = 24hr, `r604800` = 1wk |
| `sortBy` | Sort order | `DD` = Most Recent, `R` = Most Relevant |
| `keywords` | Search query | URL-encoded string |

## Verified Company ID Registry

> **Last updated**: March 13, 2026

| Company | f_C IDs | Affiliates Included | Verification Date | Notes |
|---------|---------|---------------------|-------------------|-------|
| LinkedIn | 1337,39939,2587638 | LinkedIn for Sales, LinkedIn for Marketing | 2026-03-13 | Career site (careers.linkedin.com) redirects to job board with these IDs |
| Microsoft | 1035 | — | 2026-03-13 | Also has proprietary career site at apply.careers.microsoft.com |
| DoorDash | 3205573 | — | 2026-03-13 | **CORRECTED v16.7**: Was 2520 (wrong). Verified via LinkedIn Company filter dropdown. f_C=17455 is ARC Document Solutions. |
| Meta | 10667 | — | 2026-03-13 | Also has career site at metacareers.com |

### Integration with Preferences

The pipeline also stores these IDs in `preferences.linkedin_company_filters` for runtime access. This reference file is the authoritative source — if there's a mismatch, this file wins.

## How to Find & Verify Company IDs

Company IDs are **not documented by LinkedIn** and can only be discovered through the UI:

1. Navigate to the company's LinkedIn page (e.g., `linkedin.com/company/doordash`)
2. Click the **"Jobs"** tab on the company page
3. LinkedIn redirects to a job search URL — check the `f_C=` parameter in the URL bar
4. The value after `f_C=` is the company ID (may be multiple IDs separated by commas for companies with affiliated pages)

### Verification Checklist

Before adding a new company ID to the registry:
- [ ] Navigated to company's LinkedIn page and clicked Jobs tab
- [ ] Confirmed the `f_C=` value in the resulting URL
- [ ] Ran a test search with the ID and confirmed results show the correct company name
- [ ] Checked for affiliated entities (some companies have 2-3 IDs for subsidiaries/brands)
- [ ] Updated both this file AND `preferences.linkedin_company_filters`

### Known Wrong IDs (Do Not Use)

| Wrong ID | Actually Maps To | Often Confused With |
|----------|-----------------|---------------------|
| 17455 | ARC Document Solutions | DoorDash |

> Add any future misidentified IDs here to prevent repeat mistakes.

## When to Use Company-Filtered Search

**Always use for**:
- Companies in `every_run` or `every_other` cadence that have proprietary ATS (no API)
- Priority companies where the candidate has a referral
- Companies whose career sites redirect to LinkedIn (like LinkedIn itself)

**Also use alongside standard search for**:
- Companies that may list roles under non-obvious titles (e.g., LinkedIn lists DS roles as "Applied Scientist")
- Companies where the career site search is unreliable or limited

**Do NOT use as the only channel for**:
- Companies with working ATS APIs (Greenhouse, Ashby, Lever, SmartRecruiters) — use the API as primary, LinkedIn as supplementary
- Companies with well-functioning career site search (e.g., Microsoft's apply.careers.microsoft.com) — use career site as primary

## Career Site Redirect Patterns

Some companies' career sites don't have their own search — they redirect to LinkedIn. When the pipeline encounters this pattern, it should use company-filtered LinkedIn search instead.

| Company | Career Site | Redirect Behavior |
|---------|------------|-------------------|
| LinkedIn | careers.linkedin.com | "Search Jobs" button → LinkedIn job board with f_C=1337,39939,2587638 |

> Add future redirect discoveries here.

## Maintaining This Registry

The registry should grow over time as new companies are searched:

1. **During Phase 2B** (keyword matrix): Check if each target company has an entry here
2. **During Phase 3** (channel execution): If a company has no ATS API and no entry here, discover the ID using the verification steps above
3. **After verification**: Add the new entry to both this file and `preferences.linkedin_company_filters`
4. **Quarterly audit**: Re-verify all IDs (companies occasionally restructure their LinkedIn presence)
