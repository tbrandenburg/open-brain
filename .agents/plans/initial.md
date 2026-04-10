# Refactoring Plan: open-brain Template Modernization

## Goal

Refactor this template from a Claude Code + Python scripts + hooks architecture to a
**agent-CLI-agnostic** second brain template that works with any agent CLI (opencode,
Claude Code, GitHub Copilot, Kiro, etc.) using:

- Native opencode `instructions` config for context loading (replaces SessionStart hook)
- Cron jobs calling the agent CLI in non-interactive mode (replaces Python heartbeat)
- No Python scripts anywhere in the template
- Generic paths rooted at `memory/` (no monorepo-specific paths)

---

## Repo Structure (Target)

```
open-brain/
├── AGENTS.md                      # Lean project doc - architecture only, no script paths
├── README.md                      # Rewritten setup guide (no Python, no hooks)
├── tone-of-voice.md               # Unchanged
├── .opencode/
│   └── opencode.jsonc             # NEW: project-level opencode config with instructions[]
├── memory/
│   ├── SOUL.md                    # Strip Python/hook references
│   ├── USER.md                    # Unchanged (template)
│   ├── MEMORY.md                  # Unchanged (template)
│   ├── HEARTBEAT.md               # Rewrite: cron-triggered agent CLI, checklist-only
│   ├── HABITS.md                  # Unchanged
│   ├── BOOTSTRAP.md               # Remove setup_workspace.py step
│   └── daily/
│       └── .gitkeep
└── vault/
    ├── .gitattributes             # Unchanged
    └── .gitignore                 # Unchanged
```

**Removed from template:**
- No `.claude/scripts/` directory (no Python scripts)
- No `setup_workspace.py` / `master.env` references
- No hooks configuration files

---

## Changes by File

### 1. `opencode.jsonc` (New, at project root)

Replace hook-based session context injection with OpenCode's `instructions` array.
This is the direct equivalent of the old `SessionStart` hook — memory files are loaded
as context at every session start, natively, without any script.

> Note: opencode reads config from the project root (`opencode.jsonc`), not from
> `.opencode/opencode.jsonc`. The `.opencode/` directory is used internally by opencode
> for its own state and is gitignored.

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "memory/SOUL.md",
    "memory/USER.md",
    "memory/MEMORY.md"
  ]
}
```

> Note on daily logs: `memory/daily/YYYY-MM-DD.md` cannot be auto-loaded with a dynamic
> date via glob. Convention: the agent reads today's log on first turn if it exists.

---

### 2. `AGENTS.md` (Rewrite)

Strip all Python/hook/monorepo-specific content. New content covers:

- Memory file table (SOUL, USER, MEMORY, daily) and when to update each
- Context loading: explain `instructions` in `.opencode/opencode.jsonc`
- Heartbeat: cron job calling agent CLI (`opencode run` / `claude -p` / etc.)
- Vault sync: keep git-sync section (shell script, no Python needed)

Remove entirely:
- Memory Search section (Python/FastEmbed/SQLite/Postgres)
- Daily Reflection section (Python/Agent SDK)
- Direct Platform Integrations section (Python CLI commands)
- Slack Chat Interface section
- Legacy MCP/Zapier section
- All `.claude/scripts/` path references
- All `Dynamous/` vault path references (replace with generic `memory/`)

---

### 3. `memory/SOUL.md` (Edit)

Remove:
- References to `SessionStart` hook
- "Headless/remote mode" rules mentioning the Agent SDK
- References to `heartbeat.py`, `memory_reflect.py`, `direct-integrations` skill

Update:
- "Hook injects SOUL.md at session start" → "SOUL.md is loaded via `.opencode/opencode.jsonc` instructions"

Keep:
- All personality and behavioral rules
- Memory management rules (what goes where, search before answering)
- Boundaries (ask before external comms, no data exfiltration)

---

### 4. `memory/HEARTBEAT.md` (Rewrite)

Currently documents Python `heartbeat.py` with state diffing, guardrail pre-filter,
and Agent SDK invocation. Rewrite to a simple checklist the agent follows when called
via cron — no Python dependency.

New structure:
- Cron invocation pattern (agent CLI command)
- What to check each run (email, calendar, tasks, Slack)
- Notification thresholds (immediate / batched / silent HEARTBEAT_OK)
- Draft management notes (voice-matching, send-before-confirm rule)
- Habits tracking notes

---

### 5. `memory/BOOTSTRAP.md` (Edit)

Remove:
- References to `setup_workspace.py`
- References to `settings.local.json` generation
- References to hooks setup

Update:
- Context loading step: explain `.opencode/opencode.jsonc` instructions

Keep:
- All onboarding questions (name, timezone, role, heartbeat hours, integrations)
- Post-onboarding file update steps (USER.md, SOUL.md, HEARTBEAT.md, daily log)
- Self-deletion instruction at end

---

### 6. `README.md` (Rewrite)

Current README is 716 lines tied to Python, Claude Code CLI, and a monorepo install.
New README is ~150 lines covering:

1. **Overview** — second brain template for any agent CLI
2. **Quick Start** (5 steps, no Python):
   - Clone / copy this template into your project
   - Fill in `memory/USER.md`
   - Configure your agent CLI to load context (`.opencode/opencode.jsonc` for opencode; `instructions` in other CLIs)
   - Run onboarding (BOOTSTRAP.md)
   - Optionally set up vault sync (git-sync shell script)
3. **Context Loading** — how `opencode.jsonc` injects memory files; how to adapt for other CLIs
4. **Heartbeat Setup** — cron job calling agent CLI in non-interactive mode; example cron lines
5. **Vault Sync** — condensed git-sync section
6. **Customization** — SOUL.md, HEARTBEAT.md, tone-of-voice.md

Remove entirely: Python setup, VPS/Docker, Postgres, memory search, reflection script,
Slack chat, MCP legacy.

---

## What Is NOT Changing

| File | Reason |
|------|--------|
| `tone-of-voice.md` | Standalone, no dependencies |
| `memory/USER.md` | Pure template, no references |
| `memory/MEMORY.md` | Pure template, no references |
| `memory/HABITS.md` | No Python/hook references |
| `vault/.gitattributes` | Shell-level merge driver, no scripts |
| `vault/.gitignore` | No scripts |
| `memory/daily/.gitkeep` | Placeholder |

---

## Test Criteria

The refactoring is complete and correct when:

1. `grep -r "heartbeat.py\|memory_reflect.py\|memory_search.py\|setup_workspace.py\|settings.local.json\|master.env\|SessionStart\|PreCompact\|SessionEnd\|Agent SDK\|\.claude/scripts\|Dynamous" . --include="*.md" --include="*.jsonc" --include="*.json"` returns no matches
2. `.opencode/opencode.jsonc` exists and contains a valid `instructions` array pointing to `memory/SOUL.md`, `memory/USER.md`, `memory/MEMORY.md`
3. All 3 referenced instruction files exist at those paths
4. `README.md` contains no references to Python, pip, uv, pyproject, or FastEmbed
5. `AGENTS.md` is under 100 lines
6. `memory/HEARTBEAT.md` documents a cron + agent CLI invocation pattern

---

## Status

- [x] `opencode.jsonc` created (at project root, not `.opencode/`)
- [x] `AGENTS.md` rewritten
- [x] `memory/SOUL.md` edited
- [x] `memory/HEARTBEAT.md` rewritten
- [x] `memory/BOOTSTRAP.md` edited
- [x] `README.md` rewritten
- [x] All test criteria passing
