# Decision Engine — Reasoning Requirements

Every recommendation the pipeline makes must be explainable. This file defines the reasoning structure that accompanies each Action Gate decision.

## Why Reasoning Matters

Opaque recommendations erode trust. When the user sees "HOLD" on a role they thought was a great fit, they need to understand why. Structured reasoning also helps the learning loop — when outcomes differ from predictions, we can trace which reasoning step was wrong.

## Reasoning Requirement

Each scored role MUST include a brief reasoning block explaining:

1. **Key signals detected** — The 3-5 most important signals extracted from the JD (from signal_extraction.md)
2. **Why the opportunity is attractive or unattractive** — Specific match strengths and gaps against the candidate profile
3. **Why the recommended engagement strategy is chosen** — Why this specific Action Gate and not another

## Reasoning Format

### For APPLY NOW roles:
```
REASONING: {Job Title} @ {Company}
  Signals: {seniority} {ic_vs_mgmt} in {primary_domain} | {tech_match_summary} | {company_growth}
  Attractive: {2-3 specific strengths — e.g., "direct ML team leadership matches trajectory", "GenAI focus aligns with recent skills"}
  Strategy: APPLY NOW — {warm channel available: referral from X} + {high IP: historical pattern match} + {all scores above threshold}
```

### For NETWORK FIRST roles:
```
REASONING: {Job Title} @ {Company}
  Signals: {seniority} {ic_vs_mgmt} in {primary_domain} | {tech_match_summary} | {company_growth}
  Attractive: {2-3 specific strengths}
  Gaps: {1-2 specific concerns — e.g., "no warm channel", "domain stretch"}
  Strategy: NETWORK FIRST — {scores meet threshold but no warm channel} | {networking path: moderate via alumni} | {expected timeline to convert}
```

### For HOLD roles:
```
REASONING: {Job Title} @ {Company}
  Signals: {seniority} {ic_vs_mgmt} in {primary_domain} | {tech_match_summary}
  Mixed: {what's good} vs {what's concerning}
  Strategy: HOLD — {specific reason: e.g., "IP 52 below threshold", "application cooldown active", "stale posting 45 days"} | Next: {specific action to revisit}
```

### For SKIP roles:
```
REASONING: {Job Title} @ {Company}
  Signals: {key disqualifying signals}
  Unattractive: {specific reasons — e.g., "IC-only role for management-track candidate", "PhD required", "infrastructure focus outside domain"}
  Strategy: SKIP — {primary blocker} | Revisit only if: {condition}
```

## Reasoning Constraints

Keep reasoning:
- **Concise**: 3-5 lines max per role. This is not a dissertation.
- **Specific**: Reference actual signals, not vague statements. "GenAI stack matches 4/5 requirements" not "good technical fit."
- **Actionable**: The "Strategy" line must tell the user exactly what to do and why.
- **Honest**: If the recommendation is marginal, say so. "Borderline HOLD/NETWORK FIRST — networking could tip this to actionable" is better than false confidence.

## Where Reasoning Appears

- **Action Pack**: Full reasoning block for APPLY NOW and NETWORK FIRST roles
- **CSV**: Condensed into Key Strengths (col P) and Potential Gaps (col Q) columns
- **Summary Report**: Top-level reasoning for the top 3 APPLY NOW and top 3 NETWORK FIRST roles
- **Internal**: Full reasoning available for learning loop analysis

## Reasoning as Learning Input

When outcomes arrive (interview or rejection), compare the original reasoning against the outcome:
- If a "Strong match in product DS" role gets an interview → reinforces "product DS" as a hot signal
- If a "High IP due to referral" role gets rejected → investigate whether the referral channel is weakening
- If a "Domain stretch" HOLD role later gets an interview after networking → validates the NETWORK FIRST philosophy

Store reasoning mismatches in `learning_history.reasoning_mismatches` for strategy adjustment.
