# SRED.ca Sales Coach Plugin

An automated weekly sales coaching system for SRED.ca. Powers the "John" coaching persona for Evan Batchelor's weekly VAPI voice coaching calls — from data gathering through report delivery.

## Overview

The system runs in three phases every week:

```
Monday 6:00 AM — Weekly Prep (automated)
  ↓ Pulls prior Mon-Sun data from Fireflies, HubSpot, HeyReach, Gmail
  ↓ Synthesizes Pre-Session Brief
  ↓ Injects brief into VAPI system prompt

Monday (Evan calls when ready) — Live Coaching Call
  ↓ Evan dials +1 (571) 498-9194
  ↓ 12-20 min voice session with "John" (VAPI + Claude)
  ↓ Transcript captured via Fireflies

Monday–Wednesday (automated, hourly) — Post-Session Processing
  ↓ Detects completed VAPI call transcript
  ↓ Generates Evan's Coaching Report (PDF)
  ↓ Generates Jude's Manager Summary (PDF)
  ↓ Creates Gmail draft to send Evan his report
  ↓ Updates evan-profile.md (living knowledge base)
  ↓ If no transcript by Tuesday 5pm → gentle nudge to Evan
  ↓ If no transcript by Wednesday 5pm → alert to Jude + Evan
```

## Components

### Skills

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `sales-coach-weekly-prep` | Monday 6am data pull + brief generation | Scheduled (Monday 6am) or manual prep |
| `sales-coach-post-session` | Post-call report generation + profile update | After VAPI call completes |

### Reference Files

| File | Used By | Purpose |
|------|---------|---------|
| `references/pre-session-brief-template.md` | weekly-prep, post-session | Data gathering methodology + 11-section format spec |
| `references/stage-specific-evaluation.md` | weekly-prep, post-session | Pipeline stage evaluation criteria (all 6 stages) |
| `references/sdt-coaching-framework.md` | weekly-prep, post-session | SDT ladders, Kreek frameworks, session analysis |
| `references/report-content-guide.md` | post-session | Tone rules and examples for both reports |

### Key Files in the Project Folder

| File | Purpose |
|------|---------|
| `evan-profile.md` | Living knowledge base — updated after every session |
| `evan-personal-goals.md` | Evan's personal SMART goals — PRIVATE, Evan + Coach only |
| `outputs/pre-session-brief-[DATE].txt` | Weekly data brief (generated Monday 6am) |
| `outputs/coaching-report-[DATE].pdf` | Evan's post-session PDF |
| `outputs/manager-summary-[DATE].pdf` | Jude's confidential post-session PDF |
| `outputs/vapi-prompt-assembled-[DATE].txt` | Full VAPI system prompt for manual update |

### VAPI Configuration

| Setting | Value |
|---------|-------|
| Assistant ID | `401905cf-f38f-4277-8bee-814916aaf2c0` |
| Phone | +1 (571) 498-9194 |
| Voice | ElevenLabs Josh |
| Model | Claude Sonnet 4 |
| First message | "Hey Evan, it's John. How you doing?" |

**System prompt update:** The static prompt (John's persona, methodology) lives on the assistant with `{{PRE_SESSION_BRIEF}}` and `{{PERSONAL_GOALS}}` placeholders. The weekly prep task injects fresh data via `agents/update_vapi_prompt.py` (uses `http.client` to bypass Cloudflare WAF). Requires `VAPI_API_KEY` environment variable.

## Setup

### Required Integrations

| Service | Status | Notes |
|---------|--------|-------|
| Fireflies MCP | Connected | Meeting transcripts, speaker attribution |
| HubSpot MCP | Connected | Pipeline data, email history (Evan owner ID: 228172981) |
| Gmail MCP | Connected | Jude's mailbox — for creating draft to send Evan |
| HeyReach | Browser automation | No API — Claude in Chrome at app.heyreach.io |
| VAPI | Connected | Manual system prompt update required (see above) |
| sred-doc-creator skill | Required | PDF report generation (located dynamically at runtime) |

### Scheduled Tasks

Create two scheduled tasks (or verify they exist):

1. **sales-coach-weekly-prep** — `0 6 * * 1` (Monday 6am ET)
   - Runs the weekly prep skill
   - Generates pre-session brief and assembled VAPI prompt

2. **sales-coach-post-session** — `0 * * * 1,2,3` (hourly Monday-Wednesday)
   - Polls for completed VAPI transcript via Fireflies
   - Skips if reports already exist for the current week
   - Sends a gentle email nudge to Evan at Tuesday 5pm if no call yet
   - Sends an alert to both Jude and Evan at Wednesday 5pm if still no call
   - Nudges are de-duplicated via a marker file and stop once the transcript is found

### PDF Generator Dependencies

The post-session report scripts require the `sred-doc-creator` skill:

```
Path: skills/sales-coach-post-session/scripts/
Scripts: generate_coaching_report.py, generate_manager_summary.py

Both scripts dynamically locate sred_doc.py at runtime by searching:
  1. SRED_DOC_PATH environment variable (if set)
  2. Relative paths from the script location
  3. /sessions/*/mnt/.claude/skills/sred-doc-creator/scripts/
```

Fonts (Anton, Lato) are downloaded and cached on first run. Network access required once.

## Usage

### Running the Weekly Prep Manually

Say: "Run the weekly sales coach prep" or "Generate the pre-session brief for this week."

The `sales-coach-weekly-prep` skill will activate and walk through all steps.

### Processing a Completed Call

Say: "Process this week's coaching call" or "Generate Evan's coaching report."

The `sales-coach-post-session` skill will detect the transcript and generate both reports.

### Checking Status

Say: "What's the status of this week's coaching cycle?" and Claude will check:
- Was the Monday brief generated?
- Has the VAPI call transcript appeared?
- Have the post-session reports been generated?

## Architecture Notes

- **Data window:** Always prior calendar week Monday 00:00:00 → Sunday 23:59:59 ET. Never rolling 7 days.
- **Report date:** Reports are labeled for the prior Mon-Sun window regardless of when the call happens. Even if the call is Wednesday, the report covers last Monday–Sunday.
- **evan-profile.md is the brain.** Update it after every session. It's the context that persists across weeks and months.
- **Revenue privacy:** Evan's reports never include SRED.ca company-wide revenue. Only his personal deal metrics.
- **Personal goals privacy:** Evan's personal goals (evan-personal-goals.md) never appear in anything Jude sees.
- **Escalation:** If Evan doesn't call by Tuesday 5pm, a gentle nudge goes to evan@sred.ca. If still no call by Wednesday 5pm, both jude@sred.ca and evan@sred.ca get an alert.
