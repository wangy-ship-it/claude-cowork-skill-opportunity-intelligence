# Output Formats — CSV, Action Pack, Summary Report, Google Sheet

## CSV Columns (27 columns, A-AA)

| Col | Header | Content | Format |
|-----|--------|---------|--------|
| A | Search Date | Run date | M/D/YYYY |
| B | Rank | By Interview Prob (1=highest) | Integer |
| C | Action | APPLY NOW / NETWORK FIRST / HOLD / SKIP | Text |
| D | Job Title | Full title from posting | Text |
| E | Salary Range | Min-Max base | NNNNNN-NNNNNN (no $ or commas). N/A if unknown |
| F | Location | City, State (note Remote/Hybrid) | Text |
| G | Company | Company name | Text |
| H | Team | Department or team | Text |
| I | Posted Date | Actual posting date | M/D/YYYY. Never "Recent". N/A if unknown |
| J | Fit Score | 0-100 | Integer |
| K | Opportunity Score | 0-100 | Integer |
| L | Interview Prob | 0-100 | Integer |
| M | Uplevel Risk | HIGH / MED / LOW | Text |
| N | Downlevel Risk | HIGH / MED / LOW | Text |
| O | Bucket | A / B / C | Text |
| P | Key Strengths | Match label + evidence | See below |
| Q | Potential Gaps | Specific mismatches/risks | See below |
| R | URL | Full verified URL | URL |
| S | Stage | Pipeline stage | "New" (auto-filled) |
| T | Next Action | What to do next | See below |
| U | Next Action Date | When | M/D/YYYY |
| V | Networked | Networking contact identified? | FALSE (→ checkbox) |
| W | Applied | Has applied? | FALSE (→ checkbox) |
| X | Interview | Got interview? | FALSE (→ checkbox) |
| Y | Referral | Has referral? | FALSE (→ checkbox) |
| Z | Resume Ver | Version used | v{run}_T{tier}_{company} |
| AA | JD Hash | JD snapshot filename | e.g. apple_215438_2-23-2026_1430.txt |

### Auto-filling Next Action
| Action | Stage | Next Action | Date | Networked |
|--------|-------|-------------|------|-----------|
| APPLY NOW | New | "Apply with tailored resume — see Action Pack" | Today | false |
| NETWORK FIRST | New | "Network into {Company} — find referral" | Today + 2 days | false |
| HOLD | New | "Find referral or networking contact" | Today + 3 days | false |
| SKIP | New | "Skip — revisit only if situation changes" | N/A | false |

### Key Strengths Format
Start with match label, then semicolons for specific evidence-based reasons (3-6):
> EXCELLENT MATCH: 10+ yrs analytics leadership; GenAI/LLM experience directly relevant; Cross-functional GTM collaboration matches; Snowflake/Python/SQL expertise

### Potential Gaps Format
Concrete, specific gaps. Not vague disclaimers:
> Consumer product analytics vs. B2B background; Director title may require managing managers (UPLEVEL MED)

### Output Quality Gate
- Include: APPLY NOW and HOLD only
- Exclude: SKIP where ALL THREE scores below HOLD floor (Fit <55, Opp <45, IP <40)
- Include borderline SKIPs (at least one score above HOLD floor)
- Never pad output

## Action Pack Format

Save to `{workspace}/Job_Search_Action_Pack_{M-D-YYYY}.md`

### APPLY NOW block:
```
═══════════════════════════════════════════════════
APPLY NOW: {Job Title} @ {Company}
Fit: {score} | Opportunity: {score} | Interview Prob: {score}
Cluster: {opportunity_cluster} | Path: {Strong/Moderate/Weak}
URL: {url}
═══════════════════════════════════════════════════

REASONING:
  Signals: {key extracted signals — seniority, domain, tech match summary}
  Attractive: {2-3 specific strengths}
  Strategy: APPLY NOW — {why this gate — warm channel, high scores, etc.}

TAILORED RESUME BULLETS:
1. [bullet targeting requirement X]
2. [bullet targeting requirement Y]
3. [bullet targeting requirement Z]

KEYWORD INSERTIONS:
• keyword1, keyword2, keyword3

HEADLINE TWEAK:
→ "{suggested headline}"

POSITIONING:
→ "{1-line statement}"

REASONS TO BELIEVE (Tier 1 only):
| JD Requirement | Your Proof |
|---|---|
| {req} | {proof} |

OUTREACH DRAFTS:
[Draft A: team member] [Draft B: hiring manager]

APPLICATION CHECKLIST:
☐ Update resume bullets
☐ Add keyword insertions
☐ Submit application
☐ Send outreach
☐ Log in sheet
```

### NETWORK FIRST block:
```
═══════════════════════════════════════════════════
NETWORK FIRST: {Job Title} @ {Company}
Fit: {score} | Opportunity: {score} | Interview Prob: {score}
Cluster: {opportunity_cluster} | Path: {Strong/Moderate/Weak}
URL: {url}
═══════════════════════════════════════════════════

REASONING:
  Signals: {key extracted signals}
  Attractive: {2-3 specific strengths}
  Gaps: {1-2 concerns}
  Strategy: NETWORK FIRST — {why this gate + networking plan rationale}

WHY NETWORK FIRST:
→ {reason}

REFERRAL SEARCH PLAN:
1. LinkedIn search: "{team}" "{company}" site:linkedin.com/in
2. LinkedIn search: "{alma mater}" "{company}" site:linkedin.com/in
3. Check if {existing contact} knows anyone at {company}

COFFEE CHAT OUTREACH:
[Draft]

TAILORED RESUME BULLETS (for when referral secured):
[Same format as APPLY NOW]

NETWORKING TIMELINE:
☐ Send outreach by {today+2}
☐ Follow up by {today+10} if no response
☐ Ask for referral after first conversation
☐ Cold-apply fallback: {today+14}
```

### HOLD block:
```
═══════════════════════════════════════════════════
HOLD: {Job Title} @ {Company}
Fit: {score} | Opportunity: {score} | Interview Prob: {score}
Cluster: {opportunity_cluster} | Path: {Strong/Moderate/Weak}
URL: {url}
═══════════════════════════════════════════════════

REASONING:
  Signals: {key extracted signals}
  Mixed: {what's good} vs {what's concerning}
  Strategy: HOLD — {specific reason} | Next: {specific action}

NEXT ACTION:
{what to do and when}
```

## Summary Report Template

```
═══════════════════════════════════════════════════
SEARCH SUMMARY — {M/D/YYYY}
═══════════════════════════════════════════════════

Companies searched: {list}
Previously tracked: {X}
Total relevant found: {X}
Skipped (already in sheet): {X}
NEW jobs found: {X}

ACTION BREAKDOWN:
  APPLY NOW:      {X} roles
  NETWORK FIRST:  {X} roles
  HOLD:           {X} roles
  SKIP:           {X} roles

TOP APPLY NOW:
1. {Title} @ {Company} (Fit: {X}, Opp: {X}, IP: {X}) — {reason} — {warm channel}
2. ...
3. ...

TOP NETWORK FIRST (find referrals):
1. {Title} @ {Company} (Fit: {X}) — search for: {query}
2. ...

BUCKET MIX:
  A: {X} ({%}) | B: {X} ({%}) | C: {X} ({%})

KEYWORD PERFORMANCE:
  Top: "{kw}" (confidence {X}), "{kw}" (confidence {X})
  Discovery tried: "{kw}" → avg score {X}
  Demoted: "{kw}" (confidence {X})

MARKET HEALTH:
  URL liveness: {X}% (trend: {direction})
  New roles/company: {X} avg (trend: {direction})
  Assessment: {healthy/tightening/expanding}

SEARCH QUEUE BREAKDOWN:
  Tier 1: {list} — {X} new roles
  Tier 2: {list} — {X} new roles
  Tier 3: {list} — {X} new roles

CHANNEL PERFORMANCE:
  {channel}: {apps} apps → {interviews} interviews ({rate}%) {★/⚠}
  ...
  Best channel: {channel} ({rate}%)
  Worst channel: {channel} ({rate}%)

OPPORTUNITY CLUSTERS THIS RUN:
  {cluster}: {count} roles ({action breakdown})
  ...
  Best-performing cluster historically: {cluster} ({interview_rate}%)
  Concentration check: {diversified / concentrated in {cluster}}

LEARNING APPLIED:
  Boosts: {details}
  Penalties: {details}
  Black holes: {companies}
  Cooldowns: {companies}
  Hot patterns matched: {X}
  Conversion: {X}% ({interviews}/{applied})
  Warm vs cold: warm {X}% vs cold {X}%

GUARDRAILS TRIGGERED:
  {guardrail}: {action taken} (or "None" if clean)

{If run_counter > 3:}
TREND:
  This week: {apps} apps, {screens} screens ({%})
  Last week: {apps} apps, {screens} screens ({%})
  Direction: {direction}

{Include Relationship Table — see § Relationship Table below}

{Include Interview Pipeline Dashboard — see § Interview Pipeline Dashboard below}

{Include Networking Pipeline Summary — see § Networking Pipeline Summary below}

FILES:
- CSV: {path}
- Action Pack: {path}
```

## Google Sheet — Apps Script Injection (Primary Method)

### ═══ MANDATORY: Column-to-Array Position Mapping ═══

**This mapping is the single source of truth for Apps Script data arrays.** Every row written to the sheet MUST have exactly 27 values in this exact order. This exists because Run 16's write failed due to a missing Location column (F) and extra description fields, which shifted all downstream columns and corrupted 4 rows of data.

```
Array Index → Column → Header            → Type     → Example Value
─────────────────────────────────────────────────────────────────────
[0]         → A      → Search Date        → String   → "3/7/2026"
[1]         → B      → Rank               → Integer  → 1
[2]         → C      → Action             → String   → "NETWORK FIRST"
[3]         → D      → Job Title          → String   → "Data Science Manager"
[4]         → E      → Salary Range       → String   → "200000-300000" (no $ or commas; "N/A" if unknown)
[5]         → F      → Location           → String   → "San Francisco, CA (Hybrid)"
[6]         → G      → Company            → String   → "Meta"
[7]         → H      → Team               → String   → "Reality Labs"
[8]         → I      → Posted Date        → String   → "3/1/2026" (M/D/YYYY; "N/A" if unknown)
[9]         → J      → Fit Score          → Integer  → 82
[10]        → K      → Opportunity Score   → Integer  → 78
[11]        → L      → Interview Prob      → Integer  → 65
[12]        → M      → Uplevel Risk        → String   → "LOW" / "MED" / "HIGH"
[13]        → N      → Downlevel Risk      → String   → "LOW" / "MED" / "HIGH"
[14]        → O      → Bucket              → String   → "A" / "B" / "C"
[15]        → P      → Key Strengths       → String   → "STRONG MATCH: 10+ yrs analytics; Cross-functional..."
[16]        → Q      → Potential Gaps       → String   → "Consumer vs B2B background; Team scaling needed"
[17]        → R      → URL                 → String   → "https://careers.example.com/job/12345"
[18]        → S      → Stage               → String   → "New"
[19]        → T      → Next Action         → String   → "Network into Company — find referral"
[20]        → U      → Next Action Date    → String   → "3/14/2026" (M/D/YYYY; "N/A" for SKIP)
[21]        → V      → Networked            → Boolean  → false
[22]        → W      → Applied              → Boolean  → false
[23]        → X      → Interview            → Boolean  → false
[24]        → Y      → Referral             → Boolean  → false
[25]        → Z      → Resume Ver           → String   → "" (or "v16_T1_meta")
[26]        → AA     → JD Hash              → String   → "" (or "meta_12345_3-7-2026_1430.txt")
─────────────────────────────────────────────────────────────────────
TOTAL: Exactly 27 values per row. No more, no less.
```

### ═══ COMMON MISTAKES TO AVOID ═══

These mistakes have occurred in past runs and MUST be prevented:

1. **Missing Location (F)**: Do NOT skip the Location column. Every role MUST have a location string at index [5]. If location is unknown, use "Unknown".
2. **Extra description fields**: Columns P (Key Strengths) and Q (Potential Gaps) are the ONLY two text description columns. Do NOT split descriptions into multiple columns (e.g., separate "company note", "match summary", "networking note" fields). Consolidate ALL strength/match info into one string at [15] and ALL gap/risk info into one string at [16].
3. **Scores as strings**: Fit Score [9], Opportunity Score [10], and Interview Prob [11] MUST be integers, not strings. Write `82` not `"82"`.
4. **Column count mismatch**: If your array has != 27 elements, STOP and fix it before writing. Count elements explicitly.
5. **Networking/strategy text in wrong columns**: Networking path info, strategy notes, company commentary — all go inside P (strengths) or Q (gaps) or T (next action). Never create extra columns for them.

### Writing Data

**v16.11**: The data arrays for the sheet write should come from the pre-computed csv_row arrays stored in `scored_run{N}.json` (written in Phase 8.7). These arrays were built and validated during Phase 6F while scoring context was fresh. Phase 9 reads them from the checkpoint file rather than reconstructing from memory — this is the primary reliability improvement for the sheet write.

Use Monaco API to inject Apps Script. **v16.5**: Pre-built templates are available in `scripts/` — use them instead of writing inline. See `scripts/sheet_writer_template.js` and `scripts/inject_and_run.py` for the parameterized approach. If templates are unavailable, copy the template below exactly — only replace the data values, never the structure.

**BEFORE injecting**: Read csv_row arrays from `scored_run{N}.json`. Verify the count is exactly 27 per row before adding to the data array. The Python helper `scripts/inject_and_run.py` does this validation automatically.

```javascript
// In Apps Script editor tab via javascript_tool
const editor = monaco.editor.getEditors()[0];
const code = `function addJobData() {
  var ss = SpreadsheetApp.openById('{SHEET_ID}');
  var sheet = ss.getSheetByName('New Search Feb 2026');
  var lastRow = sheet.getLastRow();
  var startRow = lastRow + 1;
  var data = [
    // EXACTLY 27 values per row. Column letters for reference:
    // [A:SearchDate, B:Rank, C:Action, D:Title, E:Salary, F:Location, G:Company, H:Team, I:Posted, J:Fit, K:Opp, L:IP, M:Uplevel, N:Downlevel, O:Bucket, P:Strengths, Q:Gaps, R:URL, S:Stage, T:NextAction, U:NextDate, V:Networked, W:Applied, X:Interview, Y:Referral, Z:ResumeVer, AA:JDHash]
    ["3/7/2026", 1, "APPLY NOW", "Data Science Manager", "200000-300000", "San Francisco, CA (Hybrid)", "Meta", "Reality Labs", "3/1/2026", 82, 78, 65, "LOW", "LOW", "A", "STRONG MATCH: analytics leadership; experimentation expertise", "Consumer vs B2B background", "https://example.com/job/123", "New", "Apply with tailored resume — see Action Pack", "3/7/2026", false, false, false, false, "", ""]
  ];
  // VALIDATION: Each row must have exactly 27 columns — abort if not
  for (var i = 0; i < data.length; i++) {
    if (data[i].length !== 27) {
      Logger.log("ERROR: Row " + i + " has " + data[i].length + " columns, expected 27. Aborting.");
      return;
    }
  }
  if (data.length > 0) {
    sheet.getRange(startRow, 1, data.length, data[0].length).setValues(data);
    // Checkboxes: V=Networked(col22), W=Applied(col23), X=Interview(col24), Y=Referral(col25)
    sheet.getRange(startRow, 22, data.length, 1).insertCheckboxes();
    sheet.getRange(startRow, 23, data.length, 1).insertCheckboxes();
    sheet.getRange(startRow, 24, data.length, 1).insertCheckboxes();
    sheet.getRange(startRow, 25, data.length, 1).insertCheckboxes();
    // Fix Posted Date format (col 9 = column I)
    sheet.getRange(startRow, 9, data.length, 1).setNumberFormat("@");
    var dates = data.map(function(r) { return [r[8]]; });
    sheet.getRange(startRow, 9, data.length, 1).setValues(dates);
    Logger.log("Added " + data.length + " rows starting at row " + startRow);
  }
}`;
editor.getModel().setValue(code);
```

**AFTER injecting, BEFORE running**: Visually count the values in each data row to confirm exactly 27. The validation code will also abort if any row has the wrong count.

Then Ctrl+S to save, ▶ to run.

### Execution Steps
1. Ctrl+S to save the script
2. Click ▶ to run
3. Authorize if prompted (first run only)
4. Proceed to Post-Write Verification Protocol below

### Conditional Formatting (FULL_ANALYSIS only)

After writing data, apply conditional formatting via a second Apps Script function. This makes the sheet scannable at a glance — green means "go", yellow means "consider", red means "concern". Without this, the sheet is a wall of numbers.

```javascript
// Inject via Monaco after the data write completes
const editor = monaco.editor.getEditors()[0];
const code = `function applyConditionalFormatting() {
  const sheet = SpreadsheetApp.getActiveSheet();
  const lastRow = sheet.getLastRow();

  // Score columns: J (Fit), K (Opp), L (IP) — green ≥75, yellow 55-74, red <55
  var scoreColumns = [10, 11, 12]; // J, K, L
  scoreColumns.forEach(function(col) {
    var range = sheet.getRange(2, col, lastRow - 1, 1);

    // Green: ≥75
    var greenRule = SpreadsheetApp.newConditionalFormatRule()
      .whenNumberGreaterThanOrEqualTo(75)
      .setBackground("#d9ead3")
      .setRanges([range])
      .build();

    // Yellow: 55-74
    var yellowRule = SpreadsheetApp.newConditionalFormatRule()
      .whenNumberBetween(55, 74)
      .setBackground("#fff2cc")
      .setRanges([range])
      .build();

    // Red: <55
    var redRule = SpreadsheetApp.newConditionalFormatRule()
      .whenNumberLessThan(55)
      .setBackground("#f4cccc")
      .setRanges([range])
      .build();

    var rules = sheet.getConditionalFormatRules();
    rules.push(greenRule, yellowRule, redRule);
    sheet.setConditionalFormatRules(rules);
  });

  // Action column C — color by gate
  var actionRange = sheet.getRange(2, 3, lastRow - 1, 1);
  var applyRule = SpreadsheetApp.newConditionalFormatRule()
    .whenTextEqualTo("APPLY NOW")
    .setBackground("#d9ead3")
    .setRanges([actionRange])
    .build();
  var networkRule = SpreadsheetApp.newConditionalFormatRule()
    .whenTextEqualTo("NETWORK FIRST")
    .setBackground("#cfe2f3")
    .setRanges([actionRange])
    .build();
  var holdRule = SpreadsheetApp.newConditionalFormatRule()
    .whenTextEqualTo("HOLD")
    .setBackground("#fff2cc")
    .setRanges([actionRange])
    .build();

  var rules = sheet.getConditionalFormatRules();
  rules.push(applyRule, networkRule, holdRule);
  sheet.setConditionalFormatRules(rules);

  Logger.log("Conditional formatting applied to " + lastRow + " rows");
}`;
editor.getModel().setValue(code);
```

Then Ctrl+S to save, ▶ to run. Check execution log for "Conditional formatting applied."

### Post-Write Verification Protocol

After writing data AND applying formatting, verify the write was successful. This step exists because Run 9's Apps Script save failed silently and the data loss wasn't caught until the next session.

1. Wait 5 seconds after the Apps Script execution completes
2. Check the execution log for "Added {X} rows starting at row {Y}"
3. **Switch to the Google Sheet tab** (not just the Apps Script tab)
4. Scroll to the last rows and take a screenshot
5. Spot-check: verify company name, Fit score, and Action of the last row match the CSV
6. If the write failed: retry once with the same script, then fall back to CSV delivery

### Two-Tab System
- **Pipeline tab**: Active roles (APPLY NOW + HOLD + recent SKIP)
- **Archive tab**: Old SKIPs, rejected, closed
- Suggest archiving when Pipeline > 80 rows

## Date Format Standard

All dates: M/D/YYYY (no leading zeros). Convert all formats encountered on career sites.

## JD Snapshot

Save to `{workspace}/jd/{company}_{jobid}_{M-D-YYYY}_{HHMM}.txt`. Store filename in CSV column Z.

## Tailoring Tiers

| Tier | Scope | Deliverables |
|------|-------|-------------|
| Tier 1 (Top 3 APPLY NOW) | Full | 3-5 bullets, 6-10 keywords, headline, positioning, Reasons-to-Believe, 2 outreach drafts, checklist |
| Tier 2 (Ranks 4-10) | Light | 2 bullets, 1 positioning, 1 outreach |

FAST_TRACK: Top 3 = Tier 1, rest = Tier 2. FULL_ANALYSIS: All APPLY NOW = Tier 1.

## Relationship Table (NEW in v14)

Include in the Summary Report. This is a logical view of CRM contacts — it does NOT modify the 27-column Google Sheet schema.

```
RELATIONSHIP TABLE — {M/D/YYYY}
═══════════════════════════════════════════════════
Name            | Company   | Strength | Status             | Follow-up Due | Linked Roles
{name}          | {company} | {str}    | {outreach_status}  | {date/—}      | {role_short}
...
═══════════════════════════════════════════════════
Active contacts: {X} | Overdue follow-ups: {X} | Referrals secured: {X}
Networking effectiveness: {top_channel} ({response_rate}%) best response rate
```

Display all contacts with `outreach_status` != `no_response` and `outreach_status` != `declined` (i.e., active contacts only). Sort by follow-up urgency (overdue first, then due today, then upcoming).

## Interview Pipeline Dashboard (NEW in v14)

Include in the Summary Report after the Relationship Table.

```
INTERVIEW PIPELINE — {M/D/YYYY}
═══════════════════════════════════════════════════

ACTIVE INTERVIEWS:
  🔵 {Company} — {Role}
     Stage: {stage} ({date})
     Prep: {X}/{Y} tasks done | Thank-you: {sent/pending/n/a}
     Next: {action by date}

PENDING RESPONSE:
  ⏳ {Company} — {Role}
     Applied: {date} | Days waiting: {X} | Referral: {contact/none}

RECENTLY CLOSED:
  {✓/✗} {Company} — {Role} — {outcome} ({date})
     {Brief learning note if rejection}

PIPELINE HEALTH:
  Active: {X} | Pending: {X} | Win rate: {X}/{Y} ({%})
  Avg days in pipeline: {X}
  Stage pass rates: Phone {X}% | Technical {X}% | Onsite {X}%
```

## Networking Pipeline Summary (NEW in v14)

Include in the Summary Report after the Interview Pipeline Dashboard.

```
NETWORKING PIPELINE — {M/D/YYYY}
═══════════════════════════════════════════════════
  DISCOVERED:            {X} roles (no networking action yet)
  CONTACT_IDENTIFIED:    {X} roles (contacts found, outreach pending)
  OUTREACH_SENT:         {X} roles (waiting for response)
  RESPONSE_RECEIVED:     {X} roles (follow-up in progress)
  REFERRAL_SECURED:      {X} roles (ready to apply or applied)
  APPLICATION_SUBMITTED: {X} roles (waiting for callback)
  INTERVIEW_STAGE:       {X} roles (active interviews)
  CLOSED:                {X} roles (completed this period)

  Conversion: DISCOVERED → CONTACT_IDENTIFIED: {X}%
  Conversion: OUTREACH_SENT → RESPONSE_RECEIVED: {X}%
  Conversion: REFERRAL_SECURED → INTERVIEW_STAGE: {X}%
```

## Google Sheet — Schema Changes (v16.9)

The schema has been expanded from 26 columns (A-Z) to 27 columns (A-AA) with the addition of the `Networked` column (V). All CRM, interview pipeline, and networking state data is stored in the preferences JSON file, NOT in the Google Sheet, but the sheet now includes the new `Networked` checkbox to track networking contact identification. The column order is: `Stage` (S), `Next Action` (T), `Next Action Date` (U), `Networked` (V), `Applied` (W), `Interview` (X), `Referral` (Y), `Resume Ver` (Z), and `JD Hash` (AA).
