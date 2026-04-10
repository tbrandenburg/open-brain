## Session Start

On the first turn of every session:
1. Check if `memory/BOOTSTRAP.md` exists.
2. If it does, read it and follow the steps in it before doing anything else.

---

## Memory Files

This project uses a `memory/` directory as persistent memory across sessions. Memory files
are auto-loaded into context via `.opencode/opencode.jsonc` — no manual reading required.

| File | Contains | Update When |
|------|----------|-------------|
| `BOOTSTRAP.md` | First-run onboarding script — guides the agent through setup questions | Never (auto-deleted after onboarding completes) |
| `SOUL.md` | AI personality, behavioral rules, communication style, boundaries | Changing how the assistant should behave |
| `USER.md` | User profile, account IDs, integration config, preferences, team info | Learning something about the user or adding an integration account |
| `MEMORY.md` | Key decisions, lessons learned, active projects, important facts | Making a significant decision or learning a reusable lesson |
| `daily/YYYY-MM-DD.md` | Session logs, heartbeat entries, daily context | End of meaningful sessions |

This file (`AGENTS.md`) is **project documentation** — how the system works and how its
components fit together. It should not contain user-specific preferences, personality rules,
or behavioral instructions. Those belong in `SOUL.md`.

> For setup and configuration instructions, see `README.md`.

### Context Loading

Memory files are loaded via the `instructions` array in `.opencode/opencode.jsonc`:

```jsonc
{
  "instructions": ["memory/SOUL.md", "memory/USER.md", "memory/MEMORY.md"]
}
```

This works with opencode out of the box. For other agent CLIs:
- **Claude Code:** use `CLAUDE.md` or `instructions` in `.claude/settings.json`
- **GitHub Copilot / Kiro:** add memory file paths to your agent context config

Daily logs (`memory/daily/YYYY-MM-DD.md`) are not auto-loaded because the date is dynamic.
Convention: on the first turn of each session, the agent reads today's log if it exists.

`memory/BOOTSTRAP.md` is not auto-loaded — the agent checks for it on session start and reads it only if it exists. It deletes itself after onboarding completes, so it is silently absent on all subsequent sessions.

---

## Heartbeat System (Proactive Second Brain)

The heartbeat is a cron job that calls the agent CLI in non-interactive mode. It runs
every 30 minutes during active hours and sends notifications when something needs attention.

### How It Works

1. Cron triggers the agent CLI with a heartbeat prompt
2. The agent reads `memory/HEARTBEAT.md` for its checklist
3. The agent uses available tools (web search, MCP, native CLI tools) to gather data
4. If something needs attention → system notification + daily log entry
5. If nothing needs attention → `HEARTBEAT_OK` logged silently

### Example Cron Setup

```bash
# Run every 30 minutes (Linux/macOS)
*/30 * * * * /path/to/run-heartbeat.sh
```

Example `run-heartbeat.sh`:
```bash
#!/bin/bash
# Adjust the command for your agent CLI:
# opencode:       opencode run "Run heartbeat. Follow memory/HEARTBEAT.md." --non-interactive
# Claude Code:    claude -p "Run heartbeat. Follow memory/HEARTBEAT.md."
# GitHub Copilot: gh copilot suggest "Run heartbeat..."
opencode run "Run heartbeat. Follow memory/HEARTBEAT.md checklist." --non-interactive
```

### Notification

After gathering data, the agent uses the OS notification tool available in its environment
(e.g., `notify-send` on Linux, `osascript` on macOS) or posts to a configured channel
(Slack, etc.). Configure your preferred output in `memory/HEARTBEAT.md`.

---

## Vault Sync (Git-Based)

The `memory/` directory syncs between machines via Git using
[simonthum/git-sync](https://github.com/simonthum/git-sync) (CC0 license) on a 2-minute timer.

### Key Files

| File | Purpose |
|------|---------|
| `vault/git-sync` | The simonthum/git-sync script (CC0, modified to use merge) |
| `vault/git-merge-concat` | Custom merge driver — concatenates append-only files instead of conflicting |
| `vault/.gitattributes` | Forces LF line endings + concat-both merge driver for daily logs |
| `vault/.gitignore` | Excludes Obsidian state, .trash/, lock files |

### How It Works

1. OS scheduler triggers git-sync every 2 minutes inside the memory repo
2. git-sync auto-commits local changes, fetches from origin, merges if diverged
3. Daily log conflicts are auto-resolved by the `concat-both` merge driver
4. If a non-daily-log conflict occurs, merge is aborted (fails gracefully, retries next cycle)

### Git Config (Required on Every Machine)

```bash
git config core.autocrlf false
git config --bool branch.main.sync true
git config --bool branch.main.syncNewFiles true

# Register the concat-both merge driver (update path to match your install)
git config merge.concat-both.name "Concatenate both sides for append-only files"
git config merge.concat-both.driver "bash /path/to/memory/vault/git-merge-concat %O %A %B"
```

See `README.md` for scheduling instructions per platform.
