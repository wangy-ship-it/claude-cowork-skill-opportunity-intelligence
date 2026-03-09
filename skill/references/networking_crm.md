# Networking CRM — Relationship Management Layer

This file defines the structured CRM system for tracking contacts, managing outreach pipelines, and automating follow-ups. The CRM transforms networking from a suggestion into a tracked, measurable workflow.

## Why a CRM Layer Matters

The existing pipeline identifies networking paths and suggests outreach. But without persistent tracking, contacts fall through the cracks, follow-ups get forgotten, and relationship-building happens ad hoc rather than systematically. This layer ensures every networking action is recorded, every contact is tracked, and every follow-up is prompted on schedule.

## Contact Record Schema

Each contact in the CRM is stored in `preferences.networking_crm.contacts[]`:

```json
{
  "contact_id": "jane-smith-stripe",
  "name": "Jane Smith",
  "company": "Stripe",
  "role_title": "Senior Data Science Manager",
  "relationship_strength": "moderate",
  "connection_source": "alumni",
  "first_contact_date": null,
  "last_contact_date": null,
  "outreach_status": "not_contacted",
  "follow_up_due_date": null,
  "linked_opportunities": ["stripe_sr_ds_mgr_12345"],
  "notes": "UIUC alum, connected on LinkedIn, posts about experimentation",
  "outreach_history": []
}
```

### Field Definitions

| Field | Type | Values | Description |
|-------|------|--------|-------------|
| `contact_id` | string | unique slug | Identifier (e.g., `jane-smith-stripe`) |
| `name` | string | — | Contact's full name |
| `company` | string | — | Current company |
| `role_title` | string | — | Their current role/title |
| `relationship_strength` | enum | `strong` / `moderate` / `weak` | How well you know this person |
| `connection_source` | enum | `alumni` / `former_colleague` / `recruiter` / `hiring_manager` / `second_degree` / `community` / `cold_outreach` | How you're connected |
| `first_contact_date` | date | M/D/YYYY or null | When you first reached out |
| `last_contact_date` | date | M/D/YYYY or null | Most recent interaction |
| `outreach_status` | enum | `not_contacted` / `message_sent` / `responded` / `referral_offered` / `referral_submitted` / `declined` / `no_response` | Current state of the relationship |
| `follow_up_due_date` | date | M/D/YYYY or null | When next action is due |
| `linked_opportunities` | array | opportunity IDs | Which roles this contact is relevant to |
| `notes` | string | — | Freeform context about the person |
| `outreach_history` | array | interaction records | Log of all interactions (see below) |

### Outreach History Entries

Each interaction with a contact is logged:

```json
{
  "date": "3/1/2026",
  "action": "message_sent",
  "channel": "linkedin_dm",
  "content_summary": "Introduced myself, referenced alumni connection, asked about DS team culture",
  "response": null,
  "next_step": "follow_up_if_no_response_by_3/8/2026"
}
```

## Relationship Strength Classification

Relationship strength is not static — it evolves based on interactions:

| Strength | Definition | How to Detect |
|----------|-----------|---------------|
| **Strong** | Former colleague, close professional contact, someone who has agreed to help | `connection_source` = `former_colleague` OR `outreach_status` = `referral_offered` or `referral_submitted` |
| **Moderate** | Alumni connection, second-degree with warm intro potential, someone who has responded positively | `connection_source` = `alumni` or `second_degree` OR `outreach_status` = `responded` |
| **Weak** | No prior relationship, cold outreach target, or unresponsive contact | Default for new contacts, or `outreach_status` = `no_response` after expiry |

### Automatic Strength Transitions

The system automatically adjusts relationship strength based on outcomes:

| Event | Transition |
|-------|-----------|
| Contact responds positively to outreach | weak → moderate, moderate stays moderate |
| Contact offers referral | any → strong |
| Contact submits referral | stays strong |
| Contact doesn't respond after 30 days | moderate → weak (expire) |
| Contact declines to help | strong → moderate, moderate → weak |
| Successful interview via contact's referral | stays strong, add to "champions" list |

## Networking Pipeline States

Every opportunity has a networking pipeline state that tracks where you are in the networking process for that role. This replaces the old approach of suggesting networking as a one-time action — now it's a tracked progression.

### States

| State | Description | Entry Condition |
|-------|-------------|-----------------|
| `DISCOVERED` | Role found, no networking action taken yet | Default for new roles |
| `CONTACT_IDENTIFIED` | A potential contact has been found for this company/role | Contact record created and linked |
| `OUTREACH_SENT` | Initial message sent to contact | `outreach_status` changed to `message_sent` |
| `RESPONSE_RECEIVED` | Contact responded (positively or neutrally) | `outreach_status` changed to `responded` |
| `REFERRAL_SECURED` | Contact agreed to refer or has submitted referral | `outreach_status` = `referral_offered` or `referral_submitted` |
| `APPLICATION_SUBMITTED` | Application sent (with or without referral) | Applied = TRUE in sheet |
| `INTERVIEW_STAGE` | Interview process is active | Interview = TRUE in sheet or stage updated |
| `CLOSED` | Process complete (hired, rejected, withdrawn, or expired) | Final disposition recorded |

### State Transitions

```
DISCOVERED
  → CONTACT_IDENTIFIED (contact found)
  → APPLICATION_SUBMITTED (cold apply, skip networking)

CONTACT_IDENTIFIED
  → OUTREACH_SENT (message sent)
  → APPLICATION_SUBMITTED (decided to cold apply instead)

OUTREACH_SENT
  → RESPONSE_RECEIVED (got reply)
  → no_response after 7 days → schedule follow-up
  → no_response after 30 days → expire, revert to DISCOVERED

RESPONSE_RECEIVED
  → REFERRAL_SECURED (contact agrees to refer)
  → OUTREACH_SENT (need another touchpoint)

REFERRAL_SECURED
  → APPLICATION_SUBMITTED (applied with referral)

APPLICATION_SUBMITTED
  → INTERVIEW_STAGE (got interview)
  → CLOSED (rejected or withdrawn)

INTERVIEW_STAGE
  → CLOSED (offer, rejection, or withdrawal)
```

### Pipeline State Storage

The networking pipeline state is tracked in the preferences file alongside each opportunity:

```json
"opportunity_pipelines": {
  "stripe_sr_ds_mgr_12345": {
    "networking_state": "OUTREACH_SENT",
    "contact_id": "jane-smith-stripe",
    "state_history": [
      {"state": "DISCOVERED", "date": "2/28/2026"},
      {"state": "CONTACT_IDENTIFIED", "date": "3/1/2026"},
      {"state": "OUTREACH_SENT", "date": "3/1/2026"}
    ],
    "last_updated": "3/1/2026"
  }
}
```

This state is NOT stored in the Google Sheet columns (the 26-column schema stays unchanged). It lives in the preferences JSON and is used internally by the pipeline to determine next actions and surface in the dashboard outputs.

## Follow-Up Automation

The CRM layer includes automated follow-up logic. During each run, the system checks all active contacts and opportunities for pending follow-ups.

### Follow-Up Rules

| Condition | Action | Timeline |
|-----------|--------|----------|
| Outreach sent, no response | Schedule follow-up message | 5-7 days after initial outreach |
| Follow-up sent, still no response | Mark as `no_response`, suggest alternative contact | 7 days after follow-up |
| Referral offered but application not submitted | Prompt user to submit application immediately | 1-2 days after referral confirmation |
| Contact responded positively | Elevate relationship strength, suggest next step (ask for referral or coffee chat) | Same day |
| Coffee chat completed | Log interaction, ask user about outcome, suggest referral ask if appropriate | 1 day after |
| Referral submitted but no interview after 14 days | Suggest checking in with contact | 14 days after submission |
| Interview completed | Prompt thank-you note (see interview_pipeline.md) | Same day or next morning |

### Follow-Up Message Templates

**First follow-up (5-7 days, no response):**
```
Hi {name}, just following up on my message from last week about the
{role} role at {company}. I know you're busy — would a 15-minute
chat work better? Happy to work around your schedule.
```

**Second follow-up rule:**
Do NOT send a second follow-up to the same contact. Instead:
1. Mark contact as `no_response`
2. Downgrade relationship strength if it was `moderate` → `weak`
3. Search for an alternative contact at the same company
4. Log: "Contact unresponsive after 2 attempts — pivoting to alternative"

**Post-referral check-in (14 days, no interview):**
```
Hi {name}, thanks again for the referral to {company}. I submitted
my application on {date}. Have you heard anything from the team?
Happy to provide any additional context that might help.
```

### Follow-Up Processing During Runs

During the networking management phase of each run:

1. Load all contacts from `preferences.networking_crm.contacts`
2. For each contact with `outreach_status` = `message_sent`:
   - Calculate days since `last_contact_date`
   - If 5-7 days and no response → add to follow-up queue
   - If 14+ days and no response → mark `no_response`, downgrade strength
3. For each contact with `outreach_status` = `referral_offered`:
   - Check linked opportunities — is application submitted?
   - If not → prompt immediate application
4. For each contact with `outreach_status` = `referral_submitted`:
   - Check if interview received (from sheet)
   - If 14+ days and no interview → suggest check-in
5. Present all pending actions in the Follow-Up Dashboard

### Follow-Up Dashboard Output

Display during Daily Check-in or Networking Mode:

```
═══════════════════════════════════════════════════
NETWORKING FOLLOW-UPS DUE
═══════════════════════════════════════════════════

OVERDUE:
  ⚠ Jane Smith @ Stripe — outreach sent 3/1, no response (8 days)
    → ACTION: Send follow-up message (template above)

DUE TODAY:
  📬 Alex Chen @ Notion — responded positively 3/3
    → ACTION: Ask about referral possibility

UPCOMING (next 3 days):
  📅 Sarah Kim @ Anthropic — follow-up due 3/10 (referral check-in)
  📅 Mike Lee @ Scale AI — outreach follow-up due 3/11

RECENTLY COMPLETED:
  ✓ Tom Zhang @ Meta — referral submitted 3/5 → application sent 3/6

PIPELINE SUMMARY:
  Active contacts: 5 | Overdue: 1 | Referrals pending: 2
  Networking pipeline: 2 OUTREACH_SENT, 1 RESPONSE_RECEIVED, 2 REFERRAL_SECURED
```

## CRM Maintenance Rules

### Across Runs

The CRM dataset persists in `preferences.networking_crm`. Each run should:

1. **Load** the existing CRM data at pipeline start (Phase 0)
2. **Check** for overdue follow-ups and state transitions (new Phase 1.5)
3. **Update** contact records based on new information (user feedback, sheet changes)
4. **Add** new contacts discovered during this run's networking evaluation
5. **Expire** stale contacts (30+ days unresponsive → downgrade strength)
6. **Save** updated CRM back to preferences (Phase 10)

### Deduplication

Before adding a new contact, check existing contacts by:
1. Name + Company match (exact)
2. Name fuzzy match + same company

### Phase 10 CRM Update Checklist

Add these items to the Phase 10 field checklist:

```
[ ] networking_crm.contacts → Add new contacts discovered this run
[ ] networking_crm.contacts → Update outreach_status for contacts acted on
[ ] networking_crm.contacts → Expire 30+ day unresponsive contacts
[ ] opportunity_pipelines → Update networking_state for each active opportunity
[ ] opportunity_pipelines → Add state_history entries for state changes
```

## Relationship Table Output

The system produces a Relationship Table as part of the Summary Report. This is a logical view displayed in the report — it does NOT modify the 26-column Google Sheet schema.

```
RELATIONSHIP TABLE — {date}
═══════════════════════════════════════════════════
Name            | Company   | Strength | Status             | Follow-up Due | Linked Roles
Jane Smith      | Stripe    | Moderate | Message Sent       | 3/8/2026      | Sr DS Mgr
Alex Chen       | Notion    | Moderate | Responded          | 3/7/2026      | Analytics Lead
Tom Zhang       | Meta      | Strong   | Referral Submitted | —             | DS Manager
Sarah Kim       | Anthropic | Moderate | Referral Offered   | 3/10/2026     | ML Lead
Mike Lee        | Scale AI  | Weak     | Message Sent       | 3/11/2026     | Data Sci Mgr
═══════════════════════════════════════════════════
Active contacts: 5 | Overdue follow-ups: 1 | Referrals secured: 2
```

This table is compatible with export to a separate CSV or spreadsheet if the user requests it, but the primary job tracking sheet remains untouched.

## How CRM Feeds Into Scoring

The CRM layer directly informs the scoring system:

1. **Networking path strength** (scoring.md) is now sourced from CRM data — a contact with `outreach_status` = `referral_offered` means Strong Path, not just "theoretical referral exists"
2. **Interview Probability** adjustments from networking boost are now evidence-based — the system knows whether a referral has actually been submitted vs. just planned
3. **Action Gate decisions** account for networking pipeline state — if outreach is already sent, the recommended action shifts from "find a contact" to "follow up" or "wait for response"
4. **Channel performance tracking** (learning_loop.md) gets more accurate data because the CRM records which channel was actually used for each opportunity
