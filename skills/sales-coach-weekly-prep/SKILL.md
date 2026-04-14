---
name: sales-coach-weekly-prep
description: >
  Run the weekly sales coach data-gathering workflow for SRED.ca. Pulls the prior week's
  (Monday–Sunday) sales activity from Fireflies, HubSpot, HeyReach, and Gmail, synthesizes a
  Pre-Session Brief, and injects it into the VAPI coaching agent system prompt. Use this skill
  whenever running the Monday 6am weekly prep task, generating a pre-session brief, or preparing
  for Evan's coaching call. Triggers on: "run the weekly prep", "generate the pre-session brief",
  "prep for Evan's call", "pull this week's data", "sales coach prep", or when the scheduled task
  invokes this workflow. This is Phase 1 of the three-phase sales coaching system.
---

# Sales Coach — Weekly Prep (Phase 1)

This skill runs every Monday at 6am ET. It gathers all of Evan's sales activity from the **prior calendar week (Monday 00:00:00 → Sunday 23:59:59 ET)**, synthesizes it into a Pre-Session Brief, and injects the brief into the VAPI coaching agent so "John" is fully briefed before Evan calls.

## Critical Date Rule

The data window is ALWAYS the **prior calendar week: Monday 00:00:00 → Sunday 23:59:59 ET**.

```
Today is Monday [DATE].
Prior week: [Last Monday] 00:00:00 ET → [Last Sunday] 23:59:59 ET

Never use "last 7 days" — always anchor to the Mon-Sun calendar week.
```

Convert to epoch milliseconds for HubSpot queries. Convert to ISO 8601 for Fireflies.

## Step 1: Read Context Files

Read these files at session start:

1. `sales-coach/CLAUDE.md` — full project context, methodology, open questions
2. `sales-coach/evan-profile.md` — living profile (last session's commitments, patterns, SDT state)
3. `sales-coach/references/pre-session-brief-template.md` — exact data format and classification logic
4. `sales-coach/references/stage-specific-evaluation.md` — pipeline stage evaluation criteria

## Step 2: Pull Data from All Four Sources

Pull all four in parallel. Every query uses the same Mon 00:00:00 → Sun 23:59:59 ET window.

### A) Fireflies — Meeting Transcripts

```
Use fireflies_get_transcripts with:
  fromDate: [prior Monday ISO 8601]
  toDate: [prior Sunday ISO 8601]

Then for each transcript involving Evan Batchelor:
  Use fireflies_get_transcript to pull full text, speaker attribution, summary, action items.

Classify each meeting:
  - Evan-led pitch/demo → high coaching value
  - Discovery/handoff (Jude or Logan present) → note who leads
  - Phone-only (no Fireflies capture) → flag as data gap

Calculate for each meeting:
  - Evan talk ratio (his lines / total lines)
  - Questions asked count
  - Follow-up email timestamp (from HubSpot, within 24hrs of meeting)
```

### B) HubSpot — Pipeline & Email Activity

**Evan's owner ID: 228172981**

```
Pull deals with activity this week:
  Filter: deals owned by 228172981
  Filter: hs_last_modified_date BETWEEN [Mon epoch ms] AND [Sun epoch ms]
  
  For each deal: stage, close date, days since last activity, deal name, amount

Pull email activity this week:
  Object type: emails
  Filter: owner_id = 228172981
  Filter: hs_timestamp BETWEEN [Mon epoch ms] AND [Sun epoch ms]
  
  For each email: subject, recipient, timestamp, was it replied to?
  Classify: personal email vs. Prospecting Agent sequence

Pipeline stage IDs:
  30153821 = Opportunity
  appointmentscheduled = SR&ED Assessment  
  31821993 = Technical Discovery
  contractsent = Follow-Up
  closedwon = Closed Won
  closedlost = Closed Lost
```

### C) HeyReach — LinkedIn Outbound

HeyReach has no API. Use browser automation (Claude in Chrome):

```
1. Navigate to app.heyreach.io
2. Log in as Jude's account
3. Filter by sender: Evan Batchelor
4. Filter by date: prior Mon-Sun
5. Pull: connection requests sent, accepted, messages sent, replied, InMails sent, replied
6. Note any active campaign names and their current stats
```

If browser automation is unavailable, note the gap and proceed without LinkedIn data.

### D) Gmail / HubSpot Email

Gmail MCP only sees Jude's mailbox. For Evan's email activity, use HubSpot (step B above). However, check Gmail for any prospect replies that may have CC'd Jude or come to the main SRED.ca inbox.

## Step 3: Check Last Week's Commitments

Read `evan-profile.md` → Commitment Tracker.

Find the most recent ⏳ Pending commitment. The VAPI prompt needs to open by asking about it.

```
Last week Evan committed to: [exact quote]
John should open: "Hey, last week you said [commitment]. How'd that go?"
```

If no prior commitments (first session), skip this step.

## Step 4: Synthesize the Pre-Session Brief

Combine all data into the Pre-Session Brief format. See `references/pre-session-brief-template.md` for the full format spec.

**Required sections:**
1. Week at a Glance (KPIs)
2. Commitment Check (last week's commitments and evidence)
3. Wins (2-4 specific things that went well)
4. Meeting Reviews (1-3 most coachable meetings with transcript excerpts)
5. Pipeline Health (stage snapshot, stalled deals flagged)
6. Payment Gate Check (any deals at Assessment where Stripe LOE is unknown)
7. Email Scorecard (personal emails, follow-up speed)
8. LinkedIn Assessment (HeyReach data for the week)
9. Patterns Observed (coach's pre-call read — what's the one thing?)
10. Coaching Focus Recommendation (what John should coach on this session)
11. SDT Notes (autonomy/competence/relatedness signals from the data)

## Step 5: Save the Brief

```
Save to: sales-coach/outputs/pre-session-brief-[YYYY-MM-DD].txt
  where YYYY-MM-DD is the prior Monday's date (the start of the reporting week)

File is plain text, ~10-20KB. This file is the anchor for the entire week's coaching cycle.
```

## Step 6: Update VAPI System Prompt

The VAPI agent needs the new brief injected. The full system prompt is ~40KB and includes:
- Static: John persona, SRED.ca context, methodology (set once, rarely changes)
- Dynamic: Pre-Session Brief + evan-profile.md snapshot (updated weekly)

**⚠ VAPI API limitation:** PATCH requests with payloads >~10KB hit Cloudflare WAF (403/1010 error). Do not attempt full system prompt update via API.

**Workaround options:**
1. (Recommended) Update only the dynamic section via API if VAPI supports partial updates
2. Manually: Jude pastes the new pre-session brief into the VAPI dashboard
   - Dashboard: https://vapi.ai
   - Assistant ID: `401905cf-f38f-4277-8bee-814916aaf2c0`
   - Find the system prompt, replace the "PRE-SESSION BRIEF" section

**The assembled prompt file:** Save the full updated prompt as:
```
sales-coach/outputs/vapi-prompt-assembled-[YYYY-MM-DD].txt
```
So Jude can paste it manually if needed.

## Step 7: Confirm and Log

Output a completion summary:

```
✅ Weekly prep complete — Week of [Monday] – [Sunday]

Data pulled:
  • Fireflies: [N] meetings found, [N] with Evan
  • HubSpot: [N] deals active, [N] emails in window
  • HeyReach: [N] connections sent, [N] messages, [N] InMails
  • Gmail: checked (no additional prospect emails found / found X relevant threads)

Pre-Session Brief: outputs/pre-session-brief-[DATE].txt ([size])
VAPI prompt: outputs/vapi-prompt-assembled-[DATE].txt

Last week's commitments in brief: [yes / none — first session]

⚠ VAPI system prompt requires manual update (Cloudflare WAF limitation).
  Paste assembled prompt from: outputs/vapi-prompt-assembled-[DATE].txt
  Dashboard: https://vapi.ai → Assistant 401905cf → System Prompt
```

## Reference Files

- `references/pre-session-brief-template.md` — exact format spec for the brief
- `references/stage-specific-evaluation.md` — pipeline stage evaluation criteria
