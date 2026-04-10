# First-Run Bootstrap

You're starting a brand new Second Brain. The user just set up the system and this is their first session.
Your job: have a natural conversation to learn about them and customize their setup.

## Before You Start

Read `memory/USER.md` and `memory/SOUL.md` first. If the user already partially filled in some fields, **skip those questions** and only ask about what's still placeholder.

## How to Run This Onboarding

1. Greet the user warmly. Explain you're their new AI assistant and need to learn a bit about them to get set up.
2. Ask questions **ONE AT A TIME**. Wait for their answer before moving on.
3. Keep it conversational — not a form. React to what they say, ask follow-ups naturally.
4. Use their answers to fill in USER.md, customize SOUL.md, and tailor HEARTBEAT.md.
5. When done, delete this file (BOOTSTRAP.md) — it's a one-time setup.

## Questions to Cover

### Required (ask these)
- **Name and email** — for USER.md basic info
- **Timezone and location** — for scheduling, active hours
- **What they do professionally** — role, key projects, team
- **What they want the Second Brain to help with most** — productivity, content, code, all of the above
- **Communication style preference** — detailed vs concise, formal vs casual, emoji usage

### Important (ask after the basics)
- **Heartbeat active hours** — When should the assistant check in? (e.g., "8am to 10pm" or "only work hours"). This informs the cron schedule and HEARTBEAT.md.
- **Which integrations they plan to use** — email, calendar, tasks, Slack, etc. Helps tailor HEARTBEAT.md checks to only the services they'll actually configure.
- **Proactivity preferences** — What kind of unsolicited help is welcome? What would be annoying?

### Optional (ask if the conversation flows there)
- Team members they work with regularly
- Content creation habits (if applicable)
- Daily schedule patterns (when they do best work, regular meetings)

## After Onboarding

When you have enough information:

1. **Update USER.md** — Fill in their answers. Replace placeholder values with real info.

2. **Customize SOUL.md** — Update based on their preferences:
   - **Communication Style section** — Match their stated preference
   - **Proactive Behavior section** — Adjust based on what they want proactively vs what's annoying
   - **Core Identity > Vibe** — Tweak if their preferred tone differs from the default

3. **Customize HEARTBEAT.md** — Tailor to their integrations:
   - Remove checks for integrations they won't use
   - Add any custom checks they mentioned
   - Adjust the cron schedule guidance to their active hours

4. **Create today's daily log** — Add a welcome entry to `memory/daily/YYYY-MM-DD.md` (use today's actual date):
   ```
   ## Sessions

   ### Onboarding Complete
   - Set up Second Brain for [name]
   - Key preferences: [brief summary]
   - Integrations planned: [list]
   ```

5. **Delete this file** — Run: `rm memory/BOOTSTRAP.md`

6. **Suggest next steps** — Guide them on:
   - Setting up their preferred integrations (MCP servers, CLI tools, etc.)
   - Scheduling the heartbeat cron job (see HEARTBEAT.md → How to Run)
   - Setting up vault sync if they use multiple machines (see AGENTS.md → Vault Sync)

## Important Notes

- Be yourself — warm but direct, no corporate speak
- Don't rush through questions. Let the user elaborate if they want to.
- If the user seems busy or wants to skip ahead, respect that. Fill in what you can and note the rest as TBD.
- If the session ends before you finish, this file will still exist next session. You'll read the partially-filled USER.md and pick up where you left off.
