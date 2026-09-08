---
name: build-pm-daily-update
description: Produce a daily build update for current programs by synthesizing connected mail, Microsoft Teams chats, and meetings/calendar.
---

# Build PM Daily Update

Use this skill when a Build PM asks for a daily build update, program status digest, build-readiness summary, or morning program review. Search the connected mail, Microsoft Teams, and meetings/calendar sources, synthesize the latest evidence for each current program, and produce one concise top-level update.

Do not invoke, depend on, or install other daily-summary skills. This skill is self-contained.

## Invocation

Accept a request such as:

> Create today's build update for my current programs. Search mail, Teams, and my meetings/calendar, call out blockers, P0 bugs, risks, decisions needed, owners, due dates, and support or escalation needed.

If the user supplies a date, reporting window, program list, or timezone, use it. Otherwise:

- Use the current date in the user's local timezone.
- Review the previous 24 hours for active developments and the previous 7 calendar days for context, decisions, and unresolved actions.
- Include only current programs discoverable from the user's sources or explicitly named by the user. Do not invent programs, owners, dates, or status.

## Source and connector assumptions

Use only connectors available in the current environment:

- **Mail:** Search inbox, sent mail, and relevant threads for program/build names, milestones, incidents, bug IDs, action terms, and decision requests. Prefer the newest message in a thread while retaining older messages when they establish a still-open commitment.
- **Microsoft Teams:** Search chats and channels, including replies and messages that mention the program, build, bug, dependency, or owner. Prefer messages with explicit status, dates, assignments, or escalation language.
- **Meetings/calendar:** Search events and meeting metadata for the reporting window and the preceding 7 days. Use titles, attendees, notes, transcripts, and follow-up actions when available. Treat an invitation or attendance alone as evidence that a meeting exists, not evidence of a decision or status.

If a connector is unavailable, say so in the **Source coverage** line and continue with the sources that are available. Never imply that an unavailable source was searched. If no source is available, return a clearly marked insufficient-evidence update rather than a fabricated summary.

## Date, time, and status handling

1. Resolve all relative dates (today, tomorrow, EOD, next week) in the user's local timezone. If the timezone is unknown, use the environment timezone and state it.
2. Preserve an explicit timezone for cross-region deadlines, for example `2026-09-09 17:00 PT`.
3. Convert date-only commitments to the user's local date for display, but do not silently convert a time-specific commitment across timezones.
4. Treat a due date as overdue only when the current local date/time has passed it.
5. Prefer explicit timestamps and calendar event times over message receipt time for commitments. When only a relative expression is present, show the normalized date and mark confidence accordingly.
6. Distinguish current state from historical state. A later correction, closure, or superseding decision replaces an earlier statement unless the newer evidence explicitly leaves the earlier item open.

## Evidence, confidence, and synthesis rules

For every material claim, retain an internal evidence record containing source type, source identifier or link when available, author/attendee, timestamp, and the supporting excerpt or structured field. Cite compact source references in the output when links are available.

Use these confidence levels:

- **High:** Explicitly stated in a recent authoritative source, or corroborated by two independent sources.
- **Medium:** Explicit but from one source, or inferred from a clear owner/date/action combination.
- **Low:** Indirect, ambiguous, stale, or based on a single unconfirmed mention.

Rules:

- Separate fact from inference. Use `Confirmed`, `Inferred`, or `Needs confirmation` language where useful.
- Do not convert silence into “on track,” and do not report a meeting as a decision unless the decision is explicit.
- Prefer the most recent authoritative owner or status update. If sources conflict, report the conflict, identify both sources, and mark the item **Needs confirmation**.
- A status label must be supported by evidence: use `Green`, `Yellow`, or `Red` only when the evidence explains why. Otherwise use `Unknown`.
- Report P0 bugs only when the source explicitly identifies P0/critical severity or clearly describes an equivalent production/build-blocking impact; do not promote ordinary bugs based on urgency alone.
- Keep blockers, risks, and decisions needed separate: a blocker prevents progress now, a risk may prevent progress, and a decision needed requires a named decision-maker or explicit approval.
- Assign an owner only when a person or team is named or unambiguously identified by the source. Preserve team ownership when no individual is named.
- Assign due dates only when explicit or safely normalized from an unambiguous relative date. Otherwise use `Not stated`.
- Include support/escalation requests only when a source requests help, identifies an escalation path, or the evidence shows that escalation is required. Name the requested action and target team/person.

## Deduplication and conflict resolution

Deduplicate before synthesizing:

- Merge repeated mentions of the same program item when they refer to the same bug, dependency, decision, or action. Use bug IDs, ticket IDs, build numbers, exact dates, and distinctive subjects to match items.
- Treat a mail thread, Teams reply chain, and meeting follow-up as potentially related evidence for one item, not three separate items.
- Keep separate items when owners, due dates, severity, build numbers, or requested outcomes differ.
- Within one source, prefer the latest message or meeting follow-up that explicitly updates the item.
- Across sources, retain the latest compatible state and preserve unresolved conflicts as a single conflict item with both citations.
- Do not count the same action, blocker, or decision multiple times in program totals.

## Procedure

1. Establish the reporting date, timezone, reporting window, and current-program scope.
2. Search mail, Teams, and meetings/calendar using program names plus build, milestone, readiness, blocker, P0, incident, dependency, risk, decision, owner, due date, escalation, and support terms.
3. Extract evidence records and normalize program names, build identifiers, bug/ticket identifiers, owners, dates, and statuses.
4. Deduplicate and reconcile updates using the rules above.
5. For each current program, classify the current status and collect blockers, P0 bugs, risks, decisions needed, owners, due dates, and support/escalation needs.
6. Rank items by severity and immediacy: P0/build blockers first, then overdue commitments, decisions needed, near-term risks, and routine updates.
7. Produce the stable output template below. Keep the top-level summary short enough to scan, while retaining enough detail and evidence to act.
8. End with source coverage and an evidence-quality note. If a program has no meaningful evidence in the reporting window, say `No material update found` rather than implying health.

## Stable output template

```markdown
# Build PM Daily Update — <YYYY-MM-DD> (<timezone>)

**Reporting window:** <start> to <end>
**Programs covered:** <program list>
**Source coverage:** Mail <searched/not available>; Teams <searched/not available>; Meetings/calendar <searched/not available>
**Overall readout:** <1-3 sentences summarizing the most important build/program movement and confidence>

## Top-level summary

| Program | Status | Latest build/milestone | Key change since prior update | Owner | Next due date | Confidence |
|---|---|---|---|---|---|---|
| <program> | <Green/Yellow/Red/Unknown> | <build or milestone> | <concise evidence-backed change> | <owner/team> | <date/time or Not stated> | <High/Medium/Low> |

## Blockers and P0 bugs

| Program | Type | Issue / impact | Owner | Due date | Immediate action or escalation | Evidence |
|---|---|---|---|---|---|---|
| <program> | <Blocker/P0> | <what is blocked or broken> | <owner> | <date or Not stated> | <action/support needed> | <source/date/link> |
| <none> | — | No confirmed blockers or P0 bugs found in the searched window. | — | — | — | — |

## Risks

| Program | Risk and likely impact | Mitigation / trigger | Owner | Review or due date | Confidence |
|---|---|---|---|---|---|
| <program> | <risk> | <mitigation or trigger> | <owner/team> | <date or Not stated> | <High/Medium/Low> |

## Decisions needed

| Program | Decision | Decision-maker | Needed by | Consequence of delay | Evidence |
|---|---|---|---|---|---|
| <program> | <decision requested> | <person/team> | <date or Not stated> | <impact> | <source/date/link> |

## Owners and due dates

| Program | Action / commitment | Owner | Due date | Status |
|---|---|---|---|---|
| <program> | <action> | <owner/team> | <date/time or Not stated> | <Open/At risk/Done/Needs confirmation> |

## Support and escalation needed

- **<Program / owner>:** <specific support or escalation request, target team/person, and needed-by date>.
- If none is evidenced, write: `No explicit support or escalation request found.`

## Evidence notes

- <Material uncertainty, source conflict, stale evidence, or missing connector note.>
- If no notable issue exists, write: `No material evidence conflicts found.`
```

## Quality bar

The finished update must be actionable without reopening every source: each nontrivial item has a program, owner or explicit `Not stated`, due date or explicit `Not stated`, current state, confidence, and a source reference when available. Keep source excerpts short, avoid copying entire messages or transcripts, and never expose unrelated personal or sensitive content.
