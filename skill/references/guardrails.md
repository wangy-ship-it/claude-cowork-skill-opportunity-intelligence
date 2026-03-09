# Guardrails

This file defines hard constraints that prevent the pipeline from recommending wasteful or counterproductive strategies. These rules override scoring outputs when violated.

## Core Philosophy

This pipeline optimizes for **strategic opportunity selection** — not application volume. Submitting 50 cold applications with 0% conversion is not productive. Spending 2 hours finding 1 strong referral is.

## Do NOT Recommend

### 1. Mass Applications to Low-Fit Roles

**Rule**: Never recommend applying to more than 3 roles per day, and never recommend applying to a role with Fit < 65.

**Why**: Low-fit applications waste time (customizing resume, writing cover letters) with near-zero return. They also dilute the candidate's "brand" — recruiters at the same company see multiple applications and interpret it as desperation.

**Enforcement**:
- If a run produces > 10 APPLY NOW roles, raise thresholds until ≤ 10 remain
- Never include a role with Fit < 65 in APPLY NOW, even if IP is high
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

## Volume vs. Quality Metrics

The pipeline tracks quality metrics, not just volume:

| Metric | Target | Anti-pattern |
|--------|--------|-------------|
| APPLY NOW per run | 3-7 | >10 (lowering standards) |
| NETWORK FIRST per run | 2-5 | 0 (ignoring networking) |
| Cold apply ratio | <30% of all applications | >50% (spray and pray) |
| Warm channel ratio | >50% of applications | <20% (underusing network) |
| Fit score of applied roles | ≥75 average | <65 average (desperation) |

Surface these in the Summary Report when they exceed anti-pattern thresholds.

## User Override

The user can override any guardrail by explicitly asking. When they do:
1. Acknowledge the override
2. Note the data that triggered the guardrail
3. Proceed with the user's request
4. Track the outcome to validate or refute the guardrail
