# Networking Strategy

This file defines how the pipeline evaluates and classifies networking paths, and how networking path strength affects scoring and engagement strategy.

## Core Principle

Warm channels convert at 80-100%. Cold channels convert at 0-5%. The most important output of any run is not "which roles to apply to" but "which networking actions to take." Every role — even APPLY NOW — should have a networking angle evaluated.

## Networking Path Strength

Every role is evaluated for networking path strength. This classification directly influences Interview Probability and the Action Gate decision.

### Strong Path

At least one of:
- **Former colleague**: Someone you've worked with directly at a previous company who is now at the target company
- **Direct hiring manager access**: You know the hiring manager personally, or have a warm intro to them
- **Trusted referral**: A close contact who can submit an internal referral and vouch for your work

**Impact on scoring**:
- Interview Probability: +20
- Action Gate: Eligible for APPLY NOW (warm channel satisfied)
- Engagement strategy: Apply immediately with referral, send personalized note to contact

**Detection**: Check `referral_tracking` in preferences. Check if any `active_pipelines` exist at the company. Check LinkedIn for 1st-degree connections at the company who are former colleagues.

### Moderate Path

At least one of:
- **Alumni connection**: Shared school, bootcamp, or professional program with someone at the target company
- **Second-degree LinkedIn connection**: A mutual connection can introduce you to someone at the company
- **Reachable team member**: Someone on the target team whose posts or content you've engaged with, or who has engaged with yours

**Impact on scoring**:
- Interview Probability: +10
- Action Gate: NETWORK FIRST (build the relationship before applying)
- Engagement strategy: Reach out to the connection first. If alumni, reference shared background. If 2nd-degree, ask mutual connection for an intro. Timeline: 1-2 weeks to convert to Strong Path.

**Detection**: Search LinkedIn for 2nd-degree connections at the company. Check alma mater matches. Check for team members who post about their work.

### Weak Path

None of the above — only:
- **Cold recruiter outreach**: No existing relationship with recruiter; would be a cold InMail or email
- **No known relationship**: No connections at the company at any degree

**Impact on scoring**:
- Interview Probability: +0 (no networking boost)
- Action Gate: NETWORK FIRST if scores are high enough, otherwise HOLD or SKIP
- Engagement strategy: Research potential contacts before applying. For Bucket A companies, cold apply may still be warranted. For Bucket B/C, invest time finding a connection first.

**Detection**: Default classification when Strong and Moderate checks both fail.

## Path Strength in the Pipeline

### Where It Runs

Path strength evaluation happens AFTER signal extraction and scoring, but BEFORE final Action Gate assignment. The flow is:

1. Extract signals (Phase 4.5)
2. Compute base scores (Fit, Opportunity, IP without networking boost)
3. **Evaluate networking path strength** (Phase 5.5)
4. Apply networking boost to IP
5. Assign final Action Gate

### How It Interacts with Action Gates

| Base Scores Meet Threshold? | Path Strength | Final Gate |
|----------------------------|---------------|------------|
| Yes (Fit ≥70, Opp ≥65, IP ≥60 after boost) | Strong | **APPLY NOW** |
| Yes | Moderate | **NETWORK FIRST** (convert to Strong, then apply) |
| Yes | Weak | **NETWORK FIRST** (find any connection) |
| Borderline (one score below threshold) | Strong | **APPLY NOW** (warm channel override) |
| Borderline | Moderate | **NETWORK FIRST** |
| Borderline | Weak | **HOLD** |
| No (scores too low) | Any | **HOLD** or **SKIP** |

### Warm Channel Override

If a role has a **Strong Path**, the Action Gate can be upgraded:
- NETWORK FIRST → APPLY NOW (if Fit ≥ 65 and IP ≥ 55 after boost)
- HOLD → NETWORK FIRST (if the warm contact is willing to advocate)

This override exists because warm channels dramatically change conversion probability.

## Referral Search Plan

For every NETWORK FIRST and HOLD role, generate a referral search plan:

```
REFERRAL SEARCH: {Company}
1. LinkedIn 1st-degree: "{company}" site:linkedin.com/in [check existing connections]
2. LinkedIn 2nd-degree: mutual connections at {company}
3. Alumni: "{alma mater}" "{company}" site:linkedin.com/in
4. Team-specific: "{team name}" "{company}" site:linkedin.com/in
5. Existing contacts: Ask {known_contacts} if they know anyone at {company}
```

## Outreach Templates by Path Strength

### Strong Path (referral request)
```
Hi {name}, I saw {Company} is hiring a {role} on the {team} team.
Given my experience in {relevant_domain}, it looks like a strong fit.
Would you be willing to submit an internal referral? Happy to send
my resume and a quick summary of why I'm a match.
```

### Moderate Path (warm intro request)
```
Hi {name}, I noticed you're at {Company} — I'm exploring a {role}
opportunity on the {team} team. We share {connection: alma mater /
mutual contact / industry group}. Would you be open to a quick chat
about the team and culture? I'd really value your perspective.
```

### Weak Path → Moderate (content engagement)
Before reaching out cold, engage with the target person's content for 1-2 weeks (like, comment thoughtfully). Then:
```
Hi {name}, I've been following your posts about {topic} — really
insightful, especially {specific_point}. I'm a {level} in {domain}
exploring opportunities in {area}. Would love to connect and learn
more about {Company}'s work in this space.
```

## Tracking

Update `referral_tracking` in preferences after each networking action. Track:
- Contact name and company
- Path strength at time of outreach
- Date of first contact
- Current status: pending / responded / referral_submitted / no_response / declined
- Expire after 30 days of no response (move to weak path)

## Integration with Networking CRM (v14)

In v14, networking strategy evaluation feeds directly into the CRM layer (see `references/networking_crm.md`). The key integration points:

### From Strategy to CRM

When Phase 5.5 evaluates networking paths:
1. **Strong Path found** → Create/update CRM contact with `relationship_strength: strong`, set `outreach_status` appropriately
2. **Moderate Path found** → Create/update CRM contact with `relationship_strength: moderate`
3. **Weak Path** → Only create CRM contact if a specific outreach target is identified

For each contact added:
- Set `linked_opportunities[]` to include the current role
- Set `connection_source` based on how the path was identified
- Initialize `outreach_status` as `not_contacted`

### From CRM to Strategy

When evaluating networking paths for returning roles (already in pipeline):
1. Check CRM first — if a contact already tracked for this company, use their `outreach_status` to determine path strength
2. A contact with `outreach_status: responded` is stronger than `not_contacted`
3. A contact with `outreach_status: referral_submitted` means Strong Path with active referral

### Pipeline State Updates

After path evaluation, update `opportunity_pipelines` state:
- Contact identified → state becomes `CONTACT_IDENTIFIED`
- User reports outreach sent → state becomes `OUTREACH_SENT`
- CRM follow-up logic (Phase 1.5) handles subsequent transitions automatically

### Referral Search Plans → CRM Actions

When generating referral search plans, also create a concrete CRM action:
```
REFERRAL SEARCH: {Company}
→ CRM ACTION: Create contact "{name}" if found, link to opportunity, set follow-up for {today+2}
```

This ensures every referral search plan results in a trackable CRM entry, not just a suggestion.
