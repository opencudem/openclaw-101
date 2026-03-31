# 10 - CLI and Reference

> Complete command reference for daily OpenClaw operations.

## What this module solves

Quick lookup for commands you use every day. No narrative, just reference.

## Gateway Commands

```bash
# Status and control
openclaw gateway status              # Check if running
openclaw gateway start               # Start the Gateway
openclaw gateway stop                # Stop the Gateway
openclaw gateway restart             # Restart the Gateway
openclaw gateway logs                # View logs
openclaw gateway logs --tail 100     # Last 100 lines
openclaw gateway logs --follow       # Stream logs

# Configuration
openclaw config get <key>            # Get config value
openclaw config set <key> <value>    # Set config value
openclaw config list                 # List all config
openclaw config reset                # Reset to defaults
```

## Channel Commands

```bash
# List channels
openclaw channel list                # Show all channels
openclaw channel status <name>       # Check channel status

# Add channels
openclaw channel add telegram --token <token>
openclaw channel add discord --token <token>
openclaw channel add slack --token <token>
openclaw channel add whatsapp        # Interactive QR pairing

# Control channels
openclaw channel pause <name>        # Pause a channel
openclaw channel resume <name>       # Resume a channel
openclaw channel remove <name>       # Remove a channel
```

## Skill Commands

```bash
# Discover
openclaw skill search <query>        # Search ClawHub
openclaw skill list                  # List installed skills
openclaw skill info <name>           # Get skill details

# Install/manage
openclaw skill install <name>        # Install from ClawHub
openclaw skill install <name>@1.2.0 # Specific version
openclaw skill update <name>         # Update skill
openclaw skill remove <name>         # Uninstall skill
openclaw skill update-all            # Update all skills
```

## Cron Commands

```bash
# Manage jobs
openclaw cron list                   # List all jobs
openclaw cron add <name>             # Add a job
  --schedule "0 9 * * *"            # Cron expression
  --command "morning briefing"        # Command to run
  --channel telegram                # Where to notify
openclaw cron remove <name>          # Remove a job
openclaw cron pause <name>           # Pause a job
openclaw cron resume <name>          # Resume a job
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `OPENCLAW_PORT` | Gateway port | 3000 |
| `OPENCLAW_LOG_LEVEL` | Logging level | info |
| `OPENCLAW_CONFIG_DIR` | Config location | ~/.openclaw |
| `OPENCLAW_PROVIDER` | AI provider | anthropic |
| `OPENAI_API_KEY` | OpenAI API key | - |
| `ANTHROPIC_API_KEY` | Anthropic API key | - |

## File Locations

```
~/.openclaw/
├── config.yaml              # Main configuration
├── .env                     # Environment variables
├── gateway.log              # Gateway logs
├── skills/                  # Custom skills
│   └── my-skill/
│       ├── SKILL.md
│       └── scripts/
├── memory/                  # Memory files
│   ├── about-me.md
│   └── daily/
├── cron.yaml                # Cron job definitions
├── secrets/                 # Sensitive data
└── backups/                 # Backup files
```

## Quick Recipes

### Restart everything fresh

```bash
openclaw gateway stop
openclaw gateway start
openclaw channel list
```

### Debug a channel

```bash
openclaw channel pause <name>
openclaw gateway logs --tail 50
openclaw channel resume <name>
```

### Update everything

```bash
npm update -g openclaw
openclaw skill update-all
openclaw gateway restart
```

### Export your setup

```bash
# Backup config and skills
tar -czf openclaw-backup-$(date +%Y%m%d).tar.gz \
  ~/.openclaw/config.yaml \
  ~/.openclaw/skills \
  ~/.openclaw/memory \
  ~/.openclaw/cron.yaml
```

### Import your setup

```bash
tar -xzf openclaw-backup-20260331.tar.gz -C ~/
openclaw gateway restart
```

## Configuration Reference

```yaml
# ~/.openclaw/config.yaml

# Gateway
port: 3000
log_level: info

# AI Provider
provider: anthropic
api_key: ${ANTHROPIC_API_KEY}
model: claude-3-5-sonnet-20241022

# Channels
channels:
  telegram:
    token: ${TELEGRAM_TOKEN}
  discord:
    token: ${DISCORD_TOKEN}
  slack:
    token: ${SLACK_TOKEN}

# Skills
skills:
  entries:
    my-custom-skill:
      path: ./skills/my-custom-skill

# Security
security:
  allowed_channels:
    - telegram:12345678
  sensitive_commands:
    - execute_shell

# Cron
cron:
  enabled: true
  timezone: UTC
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | Configuration error |
| 4 | Gateway not running |
| 5 | Permission denied |

## Related modules

- Previous: [09-security-and-ops](../09-security-and-ops/)
- Troubleshooting: [TROUBLESHOOTING.md](../../TROUBLESHOOTING.md)
- Full docs: https://docs.openclaw.ai

---

**Time to complete:** 30 minutes (reference)  
**Prerequisites:** None (use anytime)  
**Outcome:** Quick command lookup for daily operations
