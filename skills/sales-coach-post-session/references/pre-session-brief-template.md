# Pre-Session Brief Template

This is the exact format specification for the weekly pre-session brief. The brief is generated every Monday at 6am by the weekly-prep skill, and consumed by the VAPI coaching agent and the post-session skill.

The brief should be plain text, ~10-20KB. Every section is required — use "No data available" if a source was unavailable, never omit a section.

## Date Anchoring

```
PRE-SESSION BRIEF
Week of [Monday, Month Day] – [Sunday, Month Day, Year]
Generated: [Today's date and time ET]
Data window: [Prior Monday] 00:00:00 ET → [Prior Sunday] 23:59:59 ET
```

## Section 1: Week at a Glance

```
== WEEK AT A GLANCE ==

Meetings this week: [N] ([N] Fireflies-recorded, [N] phone-only/unrecorded)
Active pipeline deals: [N] across [N] stages
Follow-up speed (avg): [X.X] hours from meeting to first email
Personal emails sent: [N] (vs. [N] Prospecting Agent sequences)
LinkedIn activity: [N] connection requests, [N] InMails, [N] messages
```

## Section 2: Commitment Check

```
== COMMITMENT CHECK ==

Last week Evan committed to: "[exact verbatim quote from prior session]"

Evidence from data:
  - [Specific evidence that the commitment was met, partially met, or missed]
  - [Reference HubSpot emails, Fireflies meetings, or LinkedIn activity]

Assessment: [Met / Partially Met / Not Met / Unable to Verify]

Suggested opening: "Hey Evan, last week you said [commitment]. How'd that go?"
```

If this is the first session or no prior commitment exists, write:
```
No prior commitment on record. This is [Session N / the first session].
Suggested opening: "How's the week been? What's on your mind?"
```

## Section 3: Wins

```
== THIS WEEK'S WINS ==

1. [Specific win — reference meeting name, deal name, or action]
   Why it matters: [One sentence connecting to coaching arc or skill development]

2. [Specific win]
   Why it matters: [One sentence]

[2-4 wins. Be specific. "Good week" is not a win. "Led discovery in the Think CNC call without deferring to Logan — talk ratio was 45%" is a win.]
```

## Section 4: Meeting Reviews

```
== MEETING REVIEWS ==

--- [Meeting Name] — [Date] ---
Attendees: [List, noting who led]
Duration: [X] minutes
Evan's talk ratio: [X]%

What happened:
  [2-3 sentence summary of the meeting flow]

Coaching moments:
  - [Specific moment worth discussing — quote if possible]
  - [Another moment, if applicable]

John Barrows framework check:
  Preparation: [Evidence of prep / no evidence]
  Discovery depth: [Surface-level / Good / Excellent — with evidence]
  Urgency creation: [Did Evan create a reason to act now?]
  Follow-up: [Email sent within X hours / no follow-up detected]

---
[Repeat for 1-3 most coachable meetings. Prioritize meetings where Evan led.]
```

## Section 5: Pipeline Health

```
== PIPELINE HEALTH ==

Stage           | Deals | Details
Opportunity     | [N]   | [Deal names]
SR&ED Assessment| [N]   | [Deal names + payment gate status]
Tech Discovery  | [N]   | [Deal names + days since last activity]
Follow-Up       | [N]   | [Deal names]
Closed Won      | [N]   | [Deal names + this week only]
Closed Lost     | [N]   | [Deal names + this week only]

Stalled deals (30+ days without activity):
  - [Deal name]: [X] days since last touch. Last activity: [what it was]

New this week:
  - [Any deals added or stage changes]
```

## Section 6: Payment Gate Check

```
== PAYMENT GATE CHECK ==

Deals at SR&ED Assessment where Stripe LOE status is unknown or pending:
  - [Deal name]: [Status — LOE sent / LOE received / unknown]

Rule: Do NOT schedule Technical Discovery until Stripe LOE is received and confirmed.
```

## Section 7: Email Scorecard

```
== EMAIL SCORECARD ==

Personal emails sent by Evan: [N]
  [List subject lines and recipients if available]

Prospecting Agent sequences: [N] contacts enrolled
  [Campaign name and current stats if available]

Follow-up emails after meetings:
  [Meeting name] → follow-up sent [X] hours later / no follow-up detected

Classification rule:
  - Personal email = written by Evan, unique subject, to a specific contact
  - Sequence email = sent by Prospecting Agent, template-based, bulk
```

## Section 8: LinkedIn Assessment

```
== LINKEDIN ASSESSMENT ==

Source: HeyReach (Evan's account, prior Mon-Sun)

Connection requests sent: [N]
Connection requests accepted: [N] ([X]% acceptance rate)
Messages sent: [N]
Messages replied: [N] ([X]% reply rate)
InMails sent: [N]
InMails replied: [N] ([X]% reply rate)

Active campaigns: [Campaign names and status]

[If HeyReach data unavailable: "HeyReach data not available this week (browser automation required). Gap noted."]
```

## Section 9: Patterns Observed

```
== PATTERNS OBSERVED ==

[This is the coach's pre-call read. 2-3 observations about what the data reveals.]

Examples:
  - "Evan's talk ratio dropped from 55% to 38% this week — he's listening more."
  - "All three follow-up emails went out within 2 hours. Last week it was 6+."
  - "No personal emails to stalled deals despite last week's commitment."
  - "InMail volume is high (15 sent) but reply rate is 0%. Quantity over quality."
```

## Section 10: Coaching Focus Recommendation

```
== COACHING FOCUS RECOMMENDATION ==

Recommended focus: [ONE specific topic]

Why: [2-3 sentences explaining why this is the most impactful thing to coach on this week, based on the data above]

Suggested approach: [How John should frame it — question to ask, example to reference, skill to practice]
```

## Section 11: SDT Notes

```
== SDT NOTES ==

Autonomy signals:
  - [What Evan did without being asked / what he waited for permission on]

Competence signals:
  - [Where Evan showed skill growth / where he avoided difficulty]

Relatedness signals:
  - [Engagement level with team, coaching process, prospects]

Motivation Ladder estimate: [Level 1-4] — [Label]
  Evidence: [One sentence]

Decision Ladder estimate: [Level 1-5] — [Label]
  Evidence: [One sentence]
```
