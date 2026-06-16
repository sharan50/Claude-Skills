---
name: CEOs-office-EA-Skill
description: "Use when an executive assistant is preparing for or following up on meetings in a CEO's calendar. Triggers: uploaded meeting notes (.txt), forwarded meeting-recap emails, requests for a pre-meeting brief, requests to update a master tracker, or references to a specific meeting on the day's agenda. Produces two outputs — a structured briefing document (pre-meeting) and a validated CSV of tracker edits for human review before applying (post-meeting). Enforces strict listed-company hygiene: all references to the executive's behaviour are observational only, blackout windows are flagged, and the skill never invents action items, owners, or attendees not present in the source material. Does NOT handle travel logistics, approvals queues, calendar scheduling, or drafting messages on the executive's behalf."
---

# CEO Office EA — Meeting Loop

This skill automates the meeting half of Dhruv's EA role for BA: building pre-meeting briefs and processing post-meeting outputs into the EA Master Tracker workbook. It assumes the workspace described in `references/workspace_map.md` exists in Drive and that Dhruv operates under the listed-company hygiene rules in `references/hygiene_rules.md`.

Read in order before doing any work:
1. `references/workspace_map.md` — Drive folder/file IDs. Always re-read; IDs can change.
2. `references/hygiene_rules.md` — non-negotiables for what gets written down.
3. The workflow file for the task at hand (see below).

## Which workflow

| Trigger | Workflow |
|---|---|
| "Build me a brief for [meeting]" / "What do I need for [meeting]?" / EA names an upcoming meeting | `workflows/pre_meeting_brief.md` |
| EA uploads .txt notes / forwards a Zoom recap email / "Update the tracker from [meeting]" | `workflows/post_meeting_update.md` |
| First-time use, tabs 2–10 don't exist in the workbook yet | `workflows/workbook_scaffolding.md` |

If a request is ambiguous between pre- and post-meeting, ask once: "Pre-meeting brief or post-meeting update?"

## Output destinations

**Phase 1 (current).** CSV of edits is written to `/mnt/user-data/outputs/edits_<source>_<timestamp>.csv` and presented to the EA for eyeball-review. Brief Doc is created directly in the Briefs folder (the Drive MCP supports document creation). No edits to the workbook itself yet — the EA pastes the relevant rows after reviewing.

**Phase 2 (after Apps Script bridge is installed — see `scripts/apps_script_setup.md`).** Same CSV is uploaded to the `_pending_edits` folder in Drive. The bound Apps Script picks it up within 60 seconds and applies edits to the workbook, with reversible audit log on a hidden tab. Skill behavior is unchanged; only the destination flips.

Check which phase is active by looking at `references/workspace_map.md` → `phase` field.

## Non-negotiable rules (full version in `references/hygiene_rules.md`)

1. Refer to the CEO by their agreed short-form name or initials in every artifact this skill writes. Never spell out the full name.
2. Anything written about the CEO's behavior is **observational only**: "asked for one chart instead of six bullets" — not "hates fluff," not "was impatient," not "moody today." Same rule applies to anyone else.
3. If a date falls inside a known earnings/results blackout window (Tab 8 once it exists), flag it before writing anything that references material non-public information.
4. Skill never invents action items, deadlines, owners, or attendee names that don't appear in the source materials. If something is genuinely unclear, leave a `[CHECK]` placeholder.
5. Skill never deletes rows. Updates only overwrite specified columns; appends add new rows. Deletions are a human decision.

## CSV edit output

Format defined in `references/edit_csv_spec.md`. Every row is one operation: append, update, or append-if-missing. Generated CSV must validate against that spec before being written.

## What this skill does NOT do (yet)

- Travel logistics
- Approvals queue management
- Daily Briefing generation (separate v2 skill)
- Calendar scheduling (use Google Calendar MCP directly)
- Drafting messages on the CEO's behalf (different skill — too sensitive to bundle here)

If the EA asks for any of the above, say so and offer to handle the meeting-loop part of the request.
