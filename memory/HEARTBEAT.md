# HEARTBEAT.md - Proactive Checklist

_This file defines what to check during heartbeat runs. Called via cron by the agent CLI._

## How to Run

Schedule a cron job to call your agent CLI in non-interactive mode:

```bash
# Example cron (every 30 minutes)
*/30 * * * * /path/to/run-heartbeat.sh
```

Example `run-heartbeat.sh`:
```bash
#!/bin/bash
# Adapt to your agent CLI:
# opencode:    opencode run "Run heartbeat. Follow memory/HEARTBEAT.md." --non-interactive
# Claude Code: claude -p "Run heartbeat. Follow memory/HEARTBEAT.md."
opencode run "Run heartbeat. Follow memory/HEARTBEAT.md checklist." --non-interactive
```

Set active hours and timezone by adjusting the cron schedule or adding a time check in the script.

---

## Quick Checks (Every Heartbeat)

Use available tools (web search, MCP, shell commands) to gather data for each:

- [ ] Any urgent emails in the last 2 hours?
- [ ] Any calendar events in the next 4 hours?
- [ ] Any overdue tasks?
- [ ] Any important messages in monitored channels?

## Proactive Suggestions (Every Heartbeat)

After reviewing the data above, think about:

- [ ] What should be prioritized right now given the calendar and deadlines?
- [ ] Is there anything coming up that needs prep and doesn't have it yet?
- [ ] Are there overdue items that need a specific action (reschedule, delegate, cancel)?

## Anomaly Detection

Flag anything unusual:

- [ ] Spike in unread email (vs normal baseline)
- [ ] Cancelled or moved meetings that weren't expected
- [ ] Tasks overdue by more than a week (stale — suggest cleanup)

## Periodic Checks (Rotate Through)

### Memory Maintenance (Daily)
- [ ] Review yesterday's daily log
- [ ] Extract anything worth adding to MEMORY.md

### Project Status (1-2x daily)
- [ ] Any blocked tasks that need attention?

## When to Notify

**Notify immediately** (use `notify-send`, `osascript`, Slack, or your preferred channel):
- Urgent message from important contacts
- Calendar event starting in < 2 hours with no prep notes
- Overdue tasks

**Stay silent (log HEARTBEAT_OK to daily log):**
- Nothing new since last check
- Outside active hours unless truly urgent
- Everything is on track

## Draft Management (Optional)

If you track draft replies:

- [ ] Check `memory/drafts/active/` for pending drafts
- [ ] For each: check if you've already replied on the source platform
- [ ] If replied: move to `memory/drafts/sent/` with your actual reply text
- [ ] If draft > 24 hours old with no reply: move to `memory/drafts/expired/`

## Habits Tracking (Optional)

If you use HABITS.md:

- [ ] Read HABITS.md for today's checklist state
- [ ] Suggest specific actions for unchecked pillars based on context
- [ ] If late in day and pillars unchecked: nudge with specific suggestions

---

_Update this file to add or remove checks as your needs change._
