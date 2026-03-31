# Quick Reference

> Cheat sheet for daily OpenClaw use

## Quick Commands

| Action | Command |
|--------|---------|
| Check status | `openclaw gateway status` |
| View logs | `openclaw gateway logs --tail 50` |
| List channels | `openclaw channel list` |
| List skills | `openclaw skill list` |
| List cron jobs | `openclaw cron list` |

## Channel Quick Add

```bash
# Telegram
openclaw channel add telegram --token YOUR_BOT_TOKEN

# Discord
openclaw channel add discord --token YOUR_BOT_TOKEN

# Slack
openclaw channel add slack --token xoxb-YOUR-TOKEN

# WhatsApp
openclaw channel add whatsapp --type baileys
```

## Skill Quick Install

```bash
# Search
openclaw skill search weather

# Install
openclaw skill install weather

# Update
openclaw skill update weather

# Remove
openclaw skill remove weather
```

## Cron Quick Setup

```bash
# Daily briefing at 9 AM
openclaw cron add "morning" \
  --schedule "0 9 * * *" \
  --command "morning briefing" \
  --channel telegram

# Weekly review on Fridays at 5 PM
openclaw cron add "weekly" \
  --schedule "0 17 * * 5" \
  --command "weekly review" \
  --channel discord

# Health check every 15 minutes
openclaw cron add "health" \
  --schedule "*/15 * * * *" \
  --command "health check" \
  --channel slack
```

## File Locations

| What | Where |
|------|-------|
| Config | `~/.openclaw/config.yaml` |
| Logs | `~/.openclaw/gateway.log` |
| Skills | `~/.openclaw/skills/` |
| Memory | `~/.openclaw/memory/` |
| Cron | `~/.openclaw/cron.yaml` |
| Secrets | `~/.openclaw/secrets/` |

## Common Patterns

### Morning Routine

```
1. Gateway status check
2. Review cron job results
3. Check calendar for today
4. Respond to overnight messages
```

### Troubleshooting Flow

```
1. Check gateway status
2. View recent logs
3. Verify channel tokens
4. Restart if needed
```

### Development Flow

```
1. Create skill in ~/.openclaw/skills/
2. Write SKILL.md
3. Create scripts/
4. Test in chat
5. Iterate
```

## Chat Commands

When chatting with your agent:

| Request | Example |
|---------|---------|
| Get weather | "What's the weather in Tokyo?" |
| Check calendar | "What's on my calendar today?" |
| Set reminder | "Remind me to call mom at 3pm" |
| Create todo | "Add 'buy groceries' to my todo list" |
| Search web | "Search for OpenClaw tutorials" |
| Run skill | "Run the morning briefing workflow" |

## Environment Setup

```bash
# Add to ~/.bashrc or ~/.zshrc

# OpenClaw
export PATH="$HOME/.npm-global/bin:$PATH"

# API Keys (use your own)
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENAI_API_KEY="sk-..."
export GITHUB_TOKEN="ghp_..."
export TELEGRAM_TOKEN="..."
```

## Quick Fixes

| Problem | Fix |
|---------|-----|
| Command not found | `export PATH="$HOME/.npm-global/bin:$PATH"` |
| Port in use | `lsof -i :3000` then `kill <PID>` |
| Token expired | Regenerate in respective service |
| Skill not found | `openclaw skill list` to verify |

---

Last updated: 2026-03-31
