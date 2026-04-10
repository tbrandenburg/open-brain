# Second Brain — Personal AI Assistant Template

A template for building a proactive AI second brain that works with any agent CLI (opencode, Claude Code, GitHub Copilot, Kiro, etc.). It maintains long-term memory across sessions, runs scheduled heartbeats to check your email/calendar/tasks, and syncs memory files between machines via Git.

This is a starting point — the memory files, heartbeat checklist, and personality are all yours to customize.

---

## What's Included

| File | Purpose |
|------|---------|
| `memory/SOUL.md` | AI personality, values, behavioral guidelines |
| `memory/USER.md` | Your profile — projects, preferences, integrations |
| `memory/MEMORY.md` | Long-term memory (decisions, lessons, facts) |
| `memory/HEARTBEAT.md` | Checklist for scheduled heartbeat runs |
| `memory/HABITS.md` | Optional habit tracker |
| `memory/BOOTSTRAP.md` | One-time onboarding — deleted after first setup |
| `memory/daily/` | Daily session and heartbeat logs |
| `vault/` | Git config for multi-machine memory sync |
| `opencode.jsonc` | Loads memory files as context at session start |

---

## Quick Start

### 1. Copy this template

```bash
git clone <this-repo> my-second-brain
cd my-second-brain
```

### 2. Fill in your profile

Edit `memory/USER.md` — add your name, timezone, role, and which integrations you plan to use. Or skip this and let the onboarding do it.

### 3. Configure context loading

**opencode** — `opencode.jsonc` is already set up. Run `opencode` in this directory and context loads automatically.

**Claude Code** — add to `.claude/settings.json`:
```json
{ "instructions": ["memory/SOUL.md", "memory/USER.md", "memory/MEMORY.md"] }
```

Or create a `CLAUDE.md` that includes the memory files manually.

**Other CLIs** — consult your CLI's documentation for how to load instruction files at session start. Point it at `memory/SOUL.md`, `memory/USER.md`, and `memory/MEMORY.md`.

### 4. Run onboarding

Open a session with your agent CLI. The agent will detect `memory/BOOTSTRAP.md` and start a conversational onboarding — asking about your name, timezone, role, and preferences. It will customize the memory files and delete the bootstrap file when done.

> Prefer manual setup? Delete `memory/BOOTSTRAP.md` and edit the memory files directly.

### 5. Set up the heartbeat (optional)

Schedule a cron job to run the agent CLI in non-interactive mode:

```bash
# Create run-heartbeat.sh
cat > run-heartbeat.sh << 'EOF'
#!/bin/bash
# Adapt to your agent CLI
opencode run "Run heartbeat. Follow memory/HEARTBEAT.md checklist." --non-interactive
EOF
chmod +x run-heartbeat.sh

# Add to crontab (every 30 minutes)
(crontab -l 2>/dev/null; echo "*/30 * * * * $(pwd)/run-heartbeat.sh") | crontab -
```

See `memory/HEARTBEAT.md` for details on what the agent checks and how to configure notifications.

### 6. Set up vault sync (optional — multi-machine)

If you want memory files synced between machines, initialize the `memory/` directory as its own Git repo:

```bash
cd memory
git init
git remote add origin git@github.com:you/your-second-brain-memory.git
git add .
git commit -m "init"
git push -u origin main
```

Then schedule `vault/git-sync` to run every 2 minutes. See `AGENTS.md → Vault Sync` for full setup.

---

## How Context Loading Works

`opencode.jsonc` uses the `instructions` field to inject memory files into every session:

```jsonc
{
  "instructions": [
    "memory/SOUL.md",
    "memory/USER.md",
    "memory/MEMORY.md"
  ]
}
```

This is the opencode-native equivalent of a `SessionStart` hook — no scripts needed.

**Daily logs** (`memory/daily/YYYY-MM-DD.md`) are not auto-loaded because the date is dynamic. Convention: on the first turn of each session, ask the agent to read today's log if it exists.

---

## Customization

- **`memory/SOUL.md`** — Change the AI's personality, communication style, proactivity rules
- **`memory/HEARTBEAT.md`** — Add/remove checks, adjust notification thresholds, configure integrations
- **`tone-of-voice.md`** — Detailed writing style guide referenced by SOUL.md
- **`memory/HABITS.md`** — Set up recurring habits you want tracked

---

See `AGENTS.md` for system architecture documentation.
