# Guardrails

This file defines hard constraints that prevent the pipeline from recommending wasteful or counterproductive strategies. These rules override scoring outputs when violated.

## Core Philosophy

This pipeline optimizes for **strategic opportunity selection** — not application volume. Submitting 50 cold applications with 0% conversion is not productive. Spending 2 hours finding 1 strong referral is.

## Do NOT Recommend

### 1. Mass Applications to Low-Fit Roles

**Rule**: Never recommend applying to more than 3 roles per day, and never recommend applying to a role with Fit < 65 (or < 58 for Bucket A BORDERLINE roles — see scoring.md § Borderline Review).

**Why**: Low-fit applications waste time (customizing resume, writing cover letters) with near-zero return. They also dilute the candidate's "brand" — recruiters at the same company see multiple applications and interpret it as desperation.

**Enforcement**:
- If a run produces > 10 APPLY NOW roles, raise thresholds until ≤ 10 remain
- Never include a role with Fit < 65 in APPLY NOW (Fit < 58 for Bucket A), even if IP is high
- Bucket A roles with Fit 58-64 and Opp ≥ 75 can receive HOLD — BORDERLINE (not APPLY NOW)
- If the user explicitly asks to "apply to everything," push back with data: "Your cold application interview rate is 0% across 17 applications. Networking on 3 high-fit roles will produce better results."

### 2. Spamming Recruiters

**Rule**: Never recommend contacting more than 2 recruiters at the same company in the same week. Never recommend mass InMail campaigns.

**Why**: Recruiters talk to each other. Multiple contacts from the same candidate signal desperation and poor judgment. Quality of outreach matters more than quantity.

**Enforcement**:
- Track outreach per company per week in `referral_tracking`
- If a role's recommended action involves recruiter contact, check if another recruiter at the same company was already contacted this week
- Prefer hiring manager or team member outreach over recruiter outreach when possible

### 3. Low-Probability Opportunities with Weak Alignment

**Rule**: Do not spend Action Pack space on roles where IP < 40 AND networking path is Weak AND Fit < 70.

**Why**: These roles have three strikes against them — low conversion probability, no networking angle, and questionable fit. Time spent crafting outreach for these roles is time not spent on higher-value targets.

**Enforcement**:
- Auto-SKIP roles meeting all three conditions
- Do not include in CSV (unless borderline on one dimension)
- Note in Summary Report under "Excluded" with brief reason

### 4. Applying to Companies in Active Cooldown

**Rule**: Respect application cooldowns strictly. Do not recommend applying to a company where a cooldown is active, even if the role is excellent.

**Why**: Companies like Google and OpenAI limit application frequency. Applying during cooldown wastes the application and may flag the candidate's account.

**Enforcement**:
- Check `application_cooldown` in preferences before assigning APPLY NOW
- Downgrade to HOLD with note: "Apply after cooldown expires on {date}"
- If multiple roles at same company, pick the single best-fit and recommend only that one

### 5. Ignoring Warm Channels for Cold Applications

**Rule**: If a warm channel (referral, recruiter relationship, HM connection) exists for a role, NEVER recommend cold applying. Always route through the warm channel.

**Why**: Warm channels convert at 80-100%. Cold applications convert at 0-5%. Using a cold channel when a warm one exists is leaving a 20x advantage on the table.

**Enforcement**:
- During Action Gate assignment, always check networking path strength first
- If Strong or Moderate path exists → engagement strategy must use that path
- Cold apply is a last resort, not a default

### 6. Applying to Black Hole Companies Without Strategy Change

**Rule**: If a company has ≥5 cold applications with 0 interviews (black hole), do NOT recommend another cold application. The ONLY acceptable recommendation is NETWORK FIRST with a specific referral plan.

**Why**: Repeating a failing strategy is the definition of insanity. If cold applications don't work at a company, more cold applications won't either.

**Enforcement**:
- Check `black_hole_companies` in preferences
- For black hole companies: APPLY NOW is forbidden unless warm channel exists
- Action Gate must be NETWORK FIRST with a concrete referral search plan
- Display warning in Action Pack: "⚠ {Company} is a black hole (0/{X} cold apps converted). Do NOT cold apply. Find a referral first."

### 7. Applying to IC Roles on a Management Track (NEW in v16.5)

**Rule**: If the candidate's `trajectory_type == "leadership"`, pure IC roles CANNOT receive APPLY NOW. IC-heavy-manager roles get a Fit penalty.

**Why**: Management-track candidates applying to IC roles signals career regression. It wastes application quota on roles that either won't lead to interviews (HM sees overqualified/misaligned) or won't advance the career trajectory.

**Enforcement**:
- During signal extraction (Phase 4.5), the `ic_vs_mgmt` signal classifies each role as: IC / IC-heavy-manager / balanced / management-heavy
- If `candidate_profile.trajectory_type == "leadership"` AND `ic_vs_mgmt == "IC"` → cap gate at HOLD, note: "IC-only role for management-track candidate"
- If `ic_vs_mgmt == "IC-heavy-manager"` → -10 Fit Score + note in Potential Gaps: "IC-heavy role — limited management scope"
- Roles classified as "balanced" or "management-heavy" → no penalty from this guardrail

**Detection tips**: Look for signals like "hands-on coding 80%+", "individual contributor", "no direct reports", absence of "manage", "lead team", "hire". Conversely, "player-coach" or "40% hands-on, 60% leadership" suggests balanced/IC-heavy-manager, not pure IC.

### 8. Wasting Google Application Slots (NEW in v16.9)

**Rule**: Google limits candidates to a rolling window of active applications (typically 3-4 simultaneous, with a 90-day cooldown after rejection). Never recommend APPLY NOW for a Google role without checking the application budget. Prefer NETWORK FIRST for Google roles to preserve scarce application slots for the highest-Fit opportunities.

**Why**: Google's application system tracks all submissions. Applying to stretch roles (L7 when you're L5-L6, or roles with HIGH uplevel risk) wastes a limited slot. With a 0% cold interview rate at Google historically, every Google application should be networked first.

**Data**: Run 15 — 1 cold Google application, 0 interviews. 3 of 4 Google L6 URLs dead within days of discovery. Google's DS Manager market moves fast.

**Enforcement**:
- Check `company_intelligence.Google.application_budget.slots_available` before assigning APPLY NOW
- If `slots_available == 0` → force HOLD with note: "Google application budget exhausted — wait until {cooldown_expires}"
- If `slots_available == 1` → only the single highest-Fit Google role in the run can receive APPLY NOW; all others → NETWORK FIRST or HOLD
- If `slots_available >= 2` → normal gate logic applies, but still prefer NETWORK FIRST given 0% cold conversion rate
- HIGH uplevel risk Google roles (L7+) → ALWAYS route to NETWORK FIRST regardless of budget, unless Strong networking path exists
- Track in preferences: `company_intelligence.Google.application_budget.active_applications[]` with role, date_applied, status, and expected_cooldown_date

## Volume vs. Quality Metrics

The pipeline tracks quality metrics, not just volume:

| Metric | Target | Anti-pattern |
|--------|--------|-------------|
| APPLY NOW per run | 3-7 | >10 (lowering standards) |
| NETWORK FIRST per run | 2-5 | 0 (ignoring networking) |
| Cold apply ratio | <30% of all applications | >50% (spray and pray) |
| Warm channel ratio | >50% of applications | <20% (underusing network) |
| Fit score of applied roles | ≥75 average | <65 average (desperation) |
| Google app slots used | ≤2 per quarter | >3 on stretch roles (wasting budget) |

Surface these in the Summary Report when they exceed anti-pattern thresholds.

## User Override

The user can override any guardrail by explicitly asking. When they do:
1. Acknowledge the override
2. Note the data that triggered the guardrail
3. Proceed with the user's request
4. Track the outcome to validate or refute the guardrail
