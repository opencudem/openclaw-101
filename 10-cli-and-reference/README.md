# 10 - CLI and Reference

> Complete command reference for daily OpenClaw operations.

## What this module solves

Quick lookup for commands you use every day. No narrative, just reference.

---

## Gateway Commands

```bash
# Status and control
openclaw gateway status              # Check if running
openclaw gateway start               # Start the Gateway daemon
openclaw gateway stop                # Stop the Gateway
openclaw gateway restart             # Restart the Gateway

# Logs
openclaw gateway logs                # View logs
openclaw gateway logs --tail 100     # Last 100 lines
openclaw gateway logs --follow       # Stream logs
```

---

## Channel Commands

```bash
# List and status
openclaw channels list               # Show all channels
openclaw channels status             # Check channel status
openclaw channels capabilities       # Check what channels support

# Add channels
openclaw channels add --channel telegram --token <bot-token>
openclaw channels add --channel discord --token <bot-token>
openclaw channels add --channel slack --token <xoxb-token>
openclaw channels add --channel whatsapp  # Interactive QR pairing

# Remove channels
openclaw channels remove --channel <name> --delete

# Authentication
openclaw channels login --channel whatsapp   # QR pairing
openclaw channels logout --channel <name>    # Logout
```

---

## Pairing Commands

```bash
# List pending pairing requests
openclaw pairing list
openclaw pairing list telegram       # List for specific channel

# Approve pairing requests
openclaw pairing approve <channel> <code>
openclaw pairing approve telegram ABC123
```

---

## Skill Commands

```bash
# Discover
openclaw skills search <query>        # Search ClawHub
openclaw skills list                  # List installed skills
openclaw skills list --eligible       # List available skills
openclaw skills info <name>           # Get skill details
openclaw skills check                 # Check for skill issues

# Install/manage
openclaw skills install <slug>        # Install from ClawHub
openclaw skills install <slug> --version <version>
openclaw skills update <slug>         # Update skill
openclaw skills update --all          # Update all skills

# Remove manually from ~/.openclaw/skills/
```

---

## Agent Commands

```bash
# List and manage
openclaw agents list                  # List all agents
openclaw agents add <name>            # Add agent
openclaw agents delete <name>         # Delete agent
openclaw agents set-identity <agent> --name "Display Name"

# Bindings
openclaw agents bindings              # Show agent-channel bindings
openclaw agents bind <agent> --channel <channel>
openclaw agents unbind <agent> --channel <channel>
```

---

## Cron Commands

```bash
# List and status
openclaw cron list                    # List all jobs
openclaw cron status                  # Check cron status
openclaw cron runs --id <job-id>      # Check job run history

# Add jobs
openclaw cron add \
  --name "job-name" \
  --cron "0 9 * * *" \
  --message "What to do"

# One-shot job
openclaw cron add \
  --name "reminder" \
  --at "2026-04-05 14:00" \
  --message "Meeting in 15 minutes"

# Control jobs
openclaw cron enable <job-id>
openclaw cron disable <job-id>
openclaw cron rm <job-id>             # Remove a job
openclaw cron run <job-id>            # Run job manually

# Edit jobs
openclaw cron edit <job-id> \
  --announce \
  --channel telegram \
  --to "123456789"
```

---

## Memory Commands

```bash
openclaw memory status                # Check memory status
openclaw memory index                 # Index memory files
openclaw memory search "<query>"      # Search memory
openclaw memory search --query "<query>"
```

---

## Config Commands

```bash
# View config
openclaw config file                  # Show config file path
openclaw config schema                # Show config schema
openclaw config get <key>             # Get config value
openclaw config get                   # Get all config (pretty-printed)

# Set values
openclaw config set <key> <value>
openclaw config set browser.executablePath "/usr/bin/google-chrome"

# Set with SecretRef
openclaw config set channels.discord.token \
  --ref-provider default \
  --ref-source env \
  --ref-id DISCORD_BOT_TOKEN

# Batch set
openclaw config set --batch-file ./config.batch.json

# Unset values
openclaw config unset <key>

# Validate
openclaw config validate
openclaw config validate --json
```

---

## Doctor Commands

```bash
openclaw doctor                       # Run diagnostics
openclaw doctor --deep                # Deep check
openclaw doctor --fix                 # Fix issues automatically
```

---

## Security Commands

```bash
openclaw security audit                 # Security audit
openclaw security audit --deep        # Deep audit
openclaw security audit --fix         # Auto-fix issues
```

---

## Browser Commands

```bash
openclaw browser status               # Check browser status
openclaw browser start                # Start browser
openclaw browser stop                 # Stop browser

openclaw browser tabs                 # List tabs
openclaw browser open <url>           # Open URL
openclaw browser close <tab-id>       # Close tab

openclaw browser navigate <url>
openclaw browser screenshot
openclaw browser snapshot
openclaw browser click <selector>
openclaw browser type <selector> <text>
openclaw browser fill <selector> <value>
openclaw browser press <key>
openclaw browser wait <ms>
openclaw browser evaluate <script>
```

---

## Plugin Commands

```bash
openclaw plugins list                 # List plugins
openclaw plugins list --json          # JSON output
openclaw plugins install <spec>
openclaw plugins uninstall <plugin>
```

---

## Session Commands

```bash
openclaw sessions list                # List active sessions
openclaw sessions status              # Check session status
openclaw sessions kill <id>             # Kill a session
```

---

## Update Commands

```bash
openclaw update                       # Update to latest
openclaw update --channel stable      # stable | beta | dev
openclaw migrate                      # Migrate config after upgrade
```

---

## Global Flags

These flags work with most commands:

```bash
--dev                     # Use dev environment (separate state)
--profile <name>          # Use named profile (separate state)
--json                    # Output as JSON
--no-color                # Disable ANSI colors
--verbose                 # Verbose logging
--log-level debug         # Set log level (debug|info|warn|error)
```

---

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `OPENCLAW_PORT` | Gateway port | 18789 |
| `OPENCLAW_GATEWAY_TOKEN` | Gateway auth token | - |
| `OPENCLAW_LOG_LEVEL` | Logging level | info |
| `OPENCLAW_CONFIG_PATH` | Config file location | ~/.openclaw/openclaw.json |
| `OPENCLAW_HOME` | Base directory | ~/.openclaw |
| `OPENCLAW_SECRET_*` | Secret references | - |

### Provider API Keys (referenced via ${VAR} in config)

```bash
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
OPENROUTER_API_KEY=sk-or-...
XAI_API_KEY=...
```

---

## File Locations

```
~/.openclaw/
├── openclaw.json           # Main configuration
├── gateway.log             # Gateway logs
├── skills/                 # Custom skills
│   └── my-skill/
│       ├── SKILL.md
│       └── scripts/
├── memory/                 # Memory files
│   ├── about-me.md
│   └── daily/
├── cron/
│   └── jobs.json           # Cron job definitions
├── audit/                  # Audit logs
└── backups/                # Backup files
```

---

## Quick Recipes

### Restart everything fresh

```bash
openclaw gateway stop
openclaw gateway start
openclaw channels list
```

### Debug a channel

```bash
openclaw gateway logs --tail 50
openclaw channels status
```

### Update everything

```bash
openclaw update
openclaw skills update --all
openclaw gateway restart
```

### Export your setup

```bash
# Backup config and skills
tar -czf openclaw-backup-$(date +%Y%m%d).tar.gz \
  ~/.openclaw/openclaw.json \
  ~/.openclaw/skills \
  ~/.openclaw/memory
```

### Import your setup

```bash
tar -xzf openclaw-backup-20260331.tar.gz -C ~/
openclaw gateway restart
```

---

## Configuration Reference

```yaml
# ~/.openclaw/openclaw.json (JSON5 format)

{
  // Gateway
  gateway: {
    port: 18789,
    mode: "local",
    bind: "loopback"
  },

  // AI Models
  models: {
    mode: "merge",
    providers: {
      anthropic: {
        apiKey: "${ANTHROPIC_API_KEY}"
      }
    }
  },

  // Agents
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-5"
      },
      heartbeat: {
        every: "30m"
      }
    },
    list: []
  },

  // Channels
  channels: {
    telegram: {
      botToken: "${TELEGRAM_BOT_TOKEN}",
      dmPolicy: "pairing"
    }
  },

  // Skills
  skills: {
    entries: {}
  },

  // Cron
  cron: {
    enabled: true
  }
}
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | Configuration error |
| 4 | Gateway not running |
| 5 | Permission denied |

---

## Related modules

- Previous: [09-security-and-ops](../09-security-and-ops/)
- Troubleshooting: [TROUBLESHOOTING.md](../../TROUBLESHOOTING.md)
- Full docs: https://docs.openclaw.ai

---

**Time to complete:** 30 minutes (reference)  
**Prerequisites:** None (use anytime)  
**Outcome:** Quick command lookup for daily operations
