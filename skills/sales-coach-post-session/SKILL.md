---
name: sales-coach-post-session
description: >
  Process a completed VAPI sales coaching call for SRED.ca and generate two branded PDF reports.
  Use this skill whenever a VAPI coaching call with Evan has completed and you need to generate
  the post-session deliverables. Triggers on: "process the coaching call", "generate the coaching
  report", "make the reports", "post-session", "the call is done", "Evan's report", "manager summary",
  "sales coaching report", or any request to process a completed sales coaching session transcript.
  Also triggers as the post-session step in the weekly sales coach scheduled task. If the VAPI
  transcript for the current week has not yet been processed, this skill handles detection and
  generation automatically.
---

# Sales Coach — Post-Session Processing

After each VAPI coaching call with Evan, this skill:

1. Detects the completed call transcript from VAPI
2. Pairs it with the week's pre-session brief and evan-profile.md
3. Generates two branded PDF reports using the sred-doc-creator scripts
4. Saves Evan's report, creates a Gmail draft, and updates evan-profile.md
5. Updates the VAPI system prompt with the next week's profile snapshot

## File Locations

Read these files at the start of every post-session run:

| File | Purpose |
|------|---------|
| `sales-coach/evan-profile.md` | Living knowledge base — read first, update last |
| `sales-coach/outputs/pre-session-brief-[DATE].txt` | This week's data brief (most recent Monday) |
| `sales-coach/agents/vapi-config.md` | VAPI assistant ID, phone number, API notes |
| `sales-coach/CLAUDE.md` | Full project context and John Barrows methodology |
| `sales-coach/references/sdt-coaching-framework.md` | SDT framework for motivation analysis |

## Step 1: Detect the VAPI Transcript

Use the Fireflies MCP to find the coaching call. VAPI calls are recorded via Fireflies.

```
Search Fireflies for transcripts from this week (Monday 00:00 → Sunday 23:59 ET)
where the meeting involves Evan Batchelor.
Filter: look for calls on or after the Monday of the current week.
The VAPI call will be 12-20 minutes long from phone number +1 (571) 498-9194.
```

**Before searching:** Check if reports already exist for this week. Look for `sales-coach/outputs/coaching-report-{week_start}.pdf` where `week_start` is this week's Monday date (YYYY-MM-DD). If that file exists, log "Reports already generated for week of [DATE]. Skipping." and exit. Do not regenerate.

If no transcript is found, check the current day and time (ET) to decide the response:

**Monday, or Tuesday before 5pm ET, or Wednesday before 5pm ET:** Silent exit.
Log "No VAPI transcript found for week of [DATE]. Will retry." and exit gracefully.

**Tuesday 5pm ET (hour = 17):** Send a gentle nudge to Evan only.
First check for marker file `sales-coach/outputs/nudge-sent-{week_start}.txt` — if it already contains "Tier 1 sent", skip the send. Otherwise, create a Google Calendar event using `gcal_create_event` with `sendUpdates: "all"` — Google sends the email notification automatically:
```
Summary: "Coaching Call Reminder — Call John"
Description:
  Hey Evan,

  Just checking in — did you get a chance to hop on the coaching call
  this week? No rush if it's happening tomorrow.

  Call +1 (571) 498-9194 when ready.

  — John
Start: Tomorrow 9:00 AM ET (15-min event)
Attendees: evan@sred.ca
sendUpdates: "all"
```
After creating, append to `sales-coach/outputs/nudge-sent-{week_start}.txt`:
```
Tier 1 sent: Tuesday [date] [time] — gcal event [event_id]
```
Then exit. Do not generate reports.

**Wednesday 5pm ET (hour = 17):** Send an alert to both Jude and Evan.
First check the marker file — if it already contains "Tier 2 sent", skip. Otherwise, create a Google Calendar event with both attendees:
```
Summary: "Missing Coaching Call — Week of [Monday date]"
Description:
  Heads up — no coaching call transcript was detected this week.

  Either the call hasn't happened yet, or Fireflies didn't capture it.
  Please check and confirm.

  Evan can call +1 (571) 498-9194 anytime.

  — Sales Coach System
Start: Today 5:30 PM ET (15-min event)
Attendees: evan@sred.ca, jude@sred.ca
sendUpdates: "all"
```
After creating, append to `sales-coach/outputs/nudge-sent-{week_start}.txt`:
```
Tier 2 sent: Wednesday [date] [time] — gcal event [event_id]
```
Then exit. Do not generate reports.

**After Wednesday (Thursday–Sunday):** The polling window is closed. Exit silently.

Do not generate reports without a transcript.

If transcript found:
- Note the call date, duration, and transcript ID
- Pull the full transcript using `fireflies_get_transcript`
- Clean up: delete `sales-coach/outputs/nudge-sent-{week_start}.txt` if it exists (no longer needed)

## Step 2: Load Context

Read in parallel:
- The most recent `pre-session-brief-[DATE].txt` from `outputs/` (the Monday brief for this week)
- `evan-profile.md` (the full living profile)
- `evan-personal-goals.md` (for Evan's report only — never for manager summary)

## Step 3: Analyze the Session

Before generating reports, synthesize what happened in the call. Extract:

**Performance Data (from transcript):**
- Did Evan self-assess accurately vs. what the data showed?
- What did the coach (John) focus on?
- How did Evan respond — receptive, defensive, deflecting, energized?
- What specific moments were most significant?
- What commitment(s) did Evan make? Quote exactly in his words.
- Did anything surprise the coach — things Evan brought up that weren't in the brief?

**Metrics to carry forward (from pre-session brief):**
- Meetings this week (count, any Fireflies-recorded)
- Pipeline touches
- Follow-up speed
- Active deals + stage distribution
- LinkedIn outbound stats for the week
- Any new deals added or stage changes

**Coaching quality check:**
- Was ONE focus maintained, or did the session drift to multiple topics?
- Did John ask before telling? (SDT autonomy check)
- Grit/Grace balance — was the session appropriately direct without being discouraging?

## Step 4: Generate Evan's Coaching Report (PDF)

Run the report generator script. See `scripts/generate_coaching_report.py` for the full implementation.

**Report structure — ALWAYS use this layout:**

```
COVER: "WEEKLY COACHING REPORT" / "Week of [Monday] – [Sunday]" / "Prepared for Evan Batchelor"

PAGE 1: WEEK AT A GLANCE
  - KPI row: Meetings | Pipeline Deals | Follow-Up Speed | Active Pipeline Value
  - 3-4 sentence narrative: what kind of week it was overall

PAGE 2: THIS WEEK'S WINS
  - 2-4 win callouts using doc.win()
  - Be specific — reference real moments from the call/brief, not generic praise

PAGE 3: MEETING REVIEW
  - Cover 1-2 most coachable meetings from the week
  - Reference specific moments (Evan led discovery? Deferred? Asked good questions?)
  - Use the John Barrows framework: prep, discovery depth, urgency, follow-up
  - Keep it factual, not harsh — Evan reads this after the call

PAGE 4: PIPELINE HEALTH
  - Branded table: Stage | Deals | Key Notes
  - Highlight any stalled deals (past expected close by 30+ days)
  - Do NOT include dollar amounts tied to company revenue

PAGE 5: THIS WEEK'S FOCUS
  - The ONE thing John coached on (not two, not three — one)
  - 2-3 paragraphs on why it matters and how Evan can apply it
  - Specific, not generic: "When Logan takes over the discovery in the Think CNC call..."

PAGE 6: YOUR COMMITMENTS
  - Quote Evan's exact words from the call
  - If Evan said "I'll send three personal emails to stalled deals by Thursday" — use that
  - Sub-header: "What John Will Ask About Next Week"

PAGE 7: A NOTE ON GROWTH (Evan only — NOT in manager summary)
  - One paragraph connecting this week's coaching to Evan's longer arc
  - Reference SDT: what does this represent on his Motivation/Decision Ladder?
  - Tone: coach-to-player, not HR-to-employee. Direct and warm.
  - Can reference personal goals briefly if relevant (never quote them directly)
```

**Critical rules:**
- Never include SRED.ca company revenue figures (QuickBooks data is Jude-only)
- Quote Evan's commitments verbatim from the transcript
- The cover date range is ALWAYS the prior calendar week (Mon-Sun), not the call date
- Use SRED_EMERALD for wins, SRED_DARK_BLUE for headers

## Step 5: Generate Jude's Manager Summary (PDF)

The Manager Summary uses the same metrics as Evan's report. The difference is the narrative layer.

**Report structure:**

```
COVER: "MANAGER SUMMARY" / "Week of [Monday] – [Sunday]" / "CONFIDENTIAL — Jude Cormier"

PAGE 1: WEEK AT A GLANCE
  - Same KPI row as Evan's report (identical metrics)
  - Same pipeline table

PAGE 2: COACHING SESSION OVERVIEW
  Sub-sections:
  - "How the Session Went" — Was Evan receptive, defensive, energized, distracted?
    Reference specific moments. What surprised the coach? What came up unprompted?
  - "What the Coach Focused On" — The one focus area and why it was chosen from the data
  - "Evan's Commitments" — Same verbatim quotes as Evan's report

PAGE 3: PIPELINE REVIEW
  - Same deal table as Evan's report
  - Flag any deals with payment gate concerns (Stripe LOE not received before Tech Discovery)
  - Flag deals past expected close date
  - Revenue can be mentioned here (Jude's report — company context OK)

PAGE 4: MANAGER INSIGHT
  - What should Jude be thinking about that only a manager can act on?
  - Examples: deal support needed, resource gaps, team dynamic observations,
    anything systemic that coaching alone can't fix
  - NOT a transcript — this is signal extraction for a busy founder
  - Keep it under 300 words. Jude wants to skim this in 2 minutes.

PAGE 5: SDT PROGRESSION NOTE
  - One short paragraph on where Evan is on the Motivation and Decision Ladders
  - Any movement since last session (or since baseline)?
  - Is the coaching creating autonomy, or creating dependency?
```

**Tone:** Direct and useful. Jude wants signal, not a transcript. Write like you're giving a verbal debrief to a busy founder who cares deeply about Evan but doesn't have time for fluff.

## Step 6: Save Outputs

```python
# File naming convention — always use the Monday date of the reporting week
week_start = "YYYY-MM-DD"  # prior Monday

evan_report_path = f"sales-coach/outputs/coaching-report-{week_start}.pdf"
manager_summary_path = f"sales-coach/outputs/manager-summary-{week_start}.pdf"
```

Save both files to `sales-coach/outputs/`.

## Step 7: Create Gmail Draft for Evan's Report

Use the Gmail MCP to create a draft with Evan's coaching report attached.

```
To: evan@sred.ca (confirm from CLAUDE.md or Jude if not listed)
Subject: Your Weekly Coaching Report — Week of [Monday, Month Day]
Body:
  Hey Evan,

  Here's your coaching report from this week's call with John.

  The main thing we focused on: [one-line summary of coaching focus].

  Your commitment for this week: [Evan's verbatim commitment].

  Talk next Monday.

  — John

[Attach: coaching-report-YYYY-MM-DD.pdf]
```

**This is a DRAFT only.** Do not send. Jude reviews and sends manually.

## Step 8: Update evan-profile.md

After reports are generated and saved, update the living profile. Append — never overwrite existing content.

**Updates to make:**

1. **Update the YAML metrics block** with any new data points from this week:
   ```yaml
   as_of: "YYYY-MM-DD"
   data_window: "week of [Monday] – [Sunday]"
   # Update any metrics that changed — talk_ratio, followup_hrs, etc.
   ```

2. **Add to Commitment Tracker:**
   ```
   | [Week start] | [Exact commitment from transcript] | ⏳ Pending | — |
   ```

3. **Add to SDT Progression Log** if there was observable movement.

4. **Append to Session Log:**
   ```
   ### Session [N] — Week of [Monday Date]
   **Type:** Weekly coaching call  
   **Duration:** [X] minutes  
   **Transcript ID:** [Fireflies ID]  
   **Pre-Session Brief:** pre-session-brief-[DATE].txt  
   **Coaching Focus:** [The one thing]  
   **Key Observations:** [2-3 sentences — what stood out in the session]  
   **Evan's Self-Assessment vs. Data:** [Did they align? Where was the gap?]  
   **How Evan Showed Up:** [receptive/defensive/energized/distracted — be specific]  
   **Commitments Made:** [Verbatim]  
   **SDT Note:** [One sentence on autonomy/competence/relatedness movement]  
   ```

5. **Add new coaching intelligence observations** if the session revealed patterns not in the profile (new verbal habit noticed, new meeting behavior, new deal pattern).

## Step 9: Confirm Completion

Output a summary to Jude:

```
✅ Post-session processing complete — Week of [Monday] – [Sunday]

Reports generated:
  • Evan's Coaching Report → outputs/coaching-report-[DATE].pdf
  • Manager Summary → outputs/manager-summary-[DATE].pdf

Gmail draft created: "Your Weekly Coaching Report — Week of [date]" (ready to send to Evan)

evan-profile.md updated:
  • Commitment added to tracker
  • Session [N] logged
  • [Any SDT or metric updates]

VAPI system prompt: [updated / not updated — note if VAPI API call succeeded or needs manual update]
```

## Error Handling

- **No transcript found:** Log and exit. Do not generate reports. The scheduled task retries hourly Mon–Wed. Nudges are sent via Google Calendar events (which trigger email notifications): gentle nudge to Evan at Tuesday 5pm ET, alert to both Jude and Evan at Wednesday 5pm ET. Each nudge sends only once per week (controlled by marker file).
- **VAPI API fails on system prompt update:** Run `agents/update_vapi_prompt.py` manually, or fall back to pasting `outputs/vapi-prompt-assembled-[DATE].txt` into the VAPI dashboard.
- **Gmail MCP draft fails:** Save the draft body as a `.txt` file in `outputs/` so Jude can copy-paste.
- **Font download fails (sred_doc.py):** Check network, retry. Fonts are cached after first download.

## Reference Files

- `references/report-content-guide.md` — detailed tone guidance, example sentences, what to say vs. avoid
- `references/sdt-coaching-framework.md` — SDT framework for motivation analysis and progression tracking
- `references/stage-specific-evaluation.md` — pipeline stage evaluation criteria (for pipeline health section)
- `skills/sales-coach-post-session/scripts/generate_coaching_report.py` — ReportLab generator for Evan's report
- `skills/sales-coach-post-session/scripts/generate_manager_summary.py` — ReportLab generator for Manager Summary
