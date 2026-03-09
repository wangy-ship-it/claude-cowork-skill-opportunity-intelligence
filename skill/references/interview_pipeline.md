# Interview Pipeline Management

This file defines how the pipeline tracks active interviews, generates preparation tasks, manages thank-you notes, and provides reminders throughout the interview process. The goal is to turn interview management from a mental overhead into a systematically tracked workflow.

## Why Interview Pipeline Tracking

Once a role moves to interview stage, the work shifts from discovery and networking to preparation and execution. Without tracking, candidates forget preparation tasks, miss follow-up windows, and lose track of where they stand across multiple concurrent interview processes. This layer keeps everything organized.

## Interview Record Schema

Each active interview process is tracked in `preferences.interview_pipeline[]`:

```json
{
  "opportunity_id": "stripe_sr_ds_mgr_12345",
  "company": "Stripe",
  "role": "Senior Data Science Manager",
  "interview_stage": "phone_screen",
  "stage_history": [
    {"stage": "applied", "date": "3/1/2026", "notes": "Applied with referral from Jane Smith"},
    {"stage": "phone_screen", "date": "3/5/2026", "notes": "Scheduled with recruiter for 3/8"}
  ],
  "upcoming_interviews": [
    {
      "date": "3/8/2026",
      "time": "2:00 PM PST",
      "type": "phone_screen",
      "interviewer": "Recruiter — Alex Kim",
      "format": "30min video call",
      "prep_tasks": [
        {"task": "Research Stripe's DS org and recent product launches", "done": false},
        {"task": "Prepare 2-min elevator pitch tailored to Stripe", "done": false},
        {"task": "Review common recruiter screen questions", "done": false}
      ]
    }
  ],
  "completed_interviews": [],
  "thank_you_notes": [],
  "next_step": "Complete prep tasks before 3/8",
  "next_step_date": "3/7/2026",
  "overall_status": "active",
  "referral_contact_id": "jane-smith-stripe",
  "notes": "Jane mentioned the HM values experimentation expertise"
}
```

## Interview Stages

Track the progression through each company's interview process:

| Stage | Description | Typical Duration |
|-------|-------------|-----------------|
| `applied` | Application submitted, waiting to hear back | 1-3 weeks |
| `phone_screen` | Initial recruiter or hiring manager screen | 30-45 min |
| `technical_screen` | Technical assessment (take-home or live) | 1-2 hours |
| `onsite_round` | Full interview loop (virtual or in-person) | 4-6 hours across multiple sessions |
| `team_match` | Team matching conversations (some companies) | 1-3 calls |
| `hiring_committee` | Waiting for committee decision (Google-style) | 1-2 weeks |
| `offer_stage` | Verbal or written offer received | Negotiation window |
| `closed_accepted` | Offer accepted | — |
| `closed_rejected` | Rejected at any stage | — |
| `closed_withdrawn` | Candidate withdrew | — |

### Stage Transition Tracking

When the user reports a stage change (or when the Google Sheet is updated), update the stage history and trigger appropriate actions:

```
applied → phone_screen
  → Generate phone screen prep tasks
  → Suggest updating referral contact (if applicable)

phone_screen → technical_screen
  → Generate technical prep tasks based on JD signals
  → Review common patterns for this company's interviews

technical_screen → onsite_round
  → Generate comprehensive onsite prep
  → Research interviewers if names provided
  → Prepare questions for each session

onsite_round → offer_stage
  → Generate negotiation prep
  → Research compensation benchmarks

Any stage → closed_rejected
  → Log outcome for learning loop
  → Send thank-you note to referral contact
  → Update CRM contact with outcome
```

## Preparation Task Generation

For each interview stage, the system generates tailored preparation tasks based on the opportunity's signal block and the company's known interview patterns.

### Phone Screen Prep

```
PREP: Phone Screen — {Role} @ {Company}
Date: {date} | Duration: {format}
═══════════════════════════════════════════════════

☐ Research {Company}'s recent news, product launches, and team updates
☐ Prepare 2-minute elevator pitch tailored to {primary_domain}
☐ Review your experience narrative for {key_requirements from signal block}
☐ Prepare 3 questions about the role and team
☐ Test video/audio setup
☐ Have resume and JD open for reference

KEY SIGNALS TO HIGHLIGHT:
  • {top 3 signals from signal extraction that match this role}

POTENTIAL QUESTIONS TO PREPARE FOR:
  • "Tell me about your experience with {primary_domain}"
  • "Why are you interested in {Company}?"
  • "What's your management style?" (if leadership role)
  • "Walk me through a project where you {key_requirement}"
```

### Technical Screen Prep

```
PREP: Technical Screen — {Role} @ {Company}
Date: {date}
═══════════════════════════════════════════════════

☐ Review {Company}'s known interview format (web search for "Company DS interview")
☐ Practice {tech_direct_matches} problems/scenarios
☐ Brush up on {tech_learnable_gaps} — these may come up
☐ Prepare a case study walkthrough from your experience in {primary_domain}
☐ Review statistical fundamentals if experimentation-focused role
☐ Practice explaining technical decisions to non-technical stakeholders

TECH STACK ALIGNMENT:
  Direct matches: {list from signal block}
  Transferable: {list} — be ready to explain the mapping
  Gaps: {list} — have a learning plan ready if asked
```

### Onsite Prep

```
PREP: Onsite — {Role} @ {Company}
Date: {date} | Sessions: {count}
═══════════════════════════════════════════════════

☐ Research each interviewer (if names provided):
    • {interviewer_1}: {their role, background, topics to discuss}
    • {interviewer_2}: {their role, background}
☐ Prepare STAR stories for:
    • Leadership challenge → {suggest story from resume}
    • Technical deep dive → {suggest project}
    • Cross-functional influence → {suggest example}
    • Failure/learning moment → {suggest story}
☐ Prepare 2 thoughtful questions per interviewer
☐ Review company values and map your experience to them
☐ Plan logistics (travel, time zones, breaks)
☐ Prepare thank-you note templates (personalize after each session)
```

## Thank-You Note Management

After every interview, the system prompts thank-you notes and tracks whether they've been sent.

### Thank-You Note Timing

| Interview Type | Send Window | Format |
|---------------|-------------|--------|
| Phone screen | Same day or next morning | Email to recruiter |
| Technical screen | Within 24 hours | Email |
| Each onsite session | Within 24 hours of final session | Individual email to each interviewer |
| After referral rejection | Within 48 hours | Message to referral contact (not the company) |

### Thank-You Note Template

```
Subject: Thank you — {Role} conversation

Hi {interviewer_name},

Thank you for taking the time to speak with me about the {role}
position at {company}. I particularly enjoyed our discussion about
{specific_topic_discussed}.

{1-2 sentences connecting your experience to something discussed}

I'm excited about the opportunity to {value_you'd_bring}. Please
don't hesitate to reach out if you need any additional information.

Best regards,
{candidate_name}
```

### Thank-You Tracking

```json
{
  "interview_date": "3/8/2026",
  "interviewer": "Alex Kim",
  "thank_you_sent": false,
  "send_by": "3/9/2026",
  "template_personalized": false
}
```

The system flags unsent thank-you notes in the Follow-Up Dashboard:

```
THANK-YOU NOTES DUE:
  ⚠ Alex Kim @ Stripe — phone screen was 3/8, thank-you not sent
    → ACTION: Send personalized thank-you email today
```

## Interview Pipeline Dashboard

Display during each run as part of the unified output:

```
═══════════════════════════════════════════════════
INTERVIEW PIPELINE — {date}
═══════════════════════════════════════════════════

ACTIVE INTERVIEWS:
  🔵 Stripe — Sr DS Manager
     Stage: Phone Screen (3/8)
     Prep: 2/5 tasks done | Thank-you: pending
     Next: Complete prep by 3/7

  🟢 Meta — DS Manager
     Stage: Onsite Round (3/12-3/13)
     Prep: 0/8 tasks done | Referral: Tom Zhang
     Next: Start prep immediately

  🟡 Notion — Analytics Lead
     Stage: Technical Screen (TBD)
     Prep: waiting for scheduling
     Next: Follow up with recruiter if no date by 3/10

PENDING RESPONSE:
  ⏳ Anthropic — ML Lead
     Applied: 3/1 | Referral: Sarah Kim
     Days waiting: 6 | Expected response: 1-2 weeks

RECENTLY CLOSED:
  ❌ Google — Sr DS Manager — Rejected at onsite (3/3)
     Learning: {brief note on what to improve}
  ✅ Scale AI — DS Manager — Offer received (3/5)

PIPELINE HEALTH:
  Active: 3 | Pending: 1 | Win rate: 1/4 (25%)
  Avg days in pipeline: 12
```

## How Interview Data Feeds Back Into Learning

Interview outcomes are some of the most valuable learning signals:

1. **Stage where rejected** → Identifies weakness areas
   - Rejected at phone screen → pitch/positioning needs work
   - Rejected at technical → skill gaps or prep quality
   - Rejected at onsite → culture fit or leadership stories

2. **Company patterns** → Some companies reject at specific stages consistently
   - Track rejection_stage by company in learning_history

3. **Cluster performance** → Interview conversion by opportunity cluster
   - Updates `cluster_outcomes` in learning_loop.md

4. **Networking impact** → Did referral-backed interviews convert better?
   - Feeds into channel performance tracking

5. **Preparation correlation** → Did completing all prep tasks correlate with passing?
   - Track prep_completion_rate vs. stage_pass_rate

### Phase 10 Interview Pipeline Update Checklist

Add these items to the Phase 10 field checklist:

```
[ ] interview_pipeline → Update stage for any interviews with new outcomes
[ ] interview_pipeline → Add new entries for roles that moved to interview
[ ] interview_pipeline → Close entries for completed processes
[ ] interview_pipeline → Generate prep tasks for newly scheduled interviews
[ ] thank_you_notes → Flag any unsent thank-you notes as overdue
```

## Integration with Google Sheet

The interview pipeline does NOT add new columns to the Google Sheet. Instead:

- The `Stage` column (S) is updated to reflect interview progress (e.g., "Phone Screen", "Onsite")
- The `Interview` checkbox (W) is marked TRUE when any interview is scheduled
- The `Next Action` column (T) is updated with interview-specific next steps
- The `Next Action Date` column (U) is updated with interview/prep dates

All detailed interview tracking (prep tasks, thank-you notes, interviewer info) lives in `preferences.interview_pipeline`, not in the sheet. This preserves the 26-column schema while adding rich tracking capabilities.

## Reminders and Prompts

The system generates context-appropriate reminders:

| Trigger | Reminder |
|---------|---------|
| Interview in 2 days | "You have a {type} interview at {Company} on {date}. {X}/{Y} prep tasks complete." |
| Interview tomorrow | "Interview tomorrow! Final prep checklist: {remaining tasks}" |
| Interview today | "Interview at {time} today. Good luck! Remember to: {key tips}" |
| Interview completed, no thank-you sent | "Don't forget to send thank-you notes from yesterday's {Company} interview" |
| 7 days post-interview, no update | "It's been 7 days since your {Company} {stage}. Consider following up with the recruiter." |
| 14 days post-interview, no update | "Still no update from {Company} after 14 days. Follow up or mark as stale." |
