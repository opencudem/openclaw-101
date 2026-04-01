# 09 - Security and Ops

> Secure your OpenClaw deployment with DM policies, pairing, and security auditing.

## What this module solves

Powerful automation needs guardrails. This module covers OpenClaw's built-in security features: DM policies, pairing, security audits, and operational best practices.

## When to use this module

- You're running OpenClaw in production
- You handle sensitive data
- You need to control who can message your agent
- You want to audit your deployment

## Security Layers

```
┌─────────────────────────────────────────┐
│           PERIMETER                     │
│  - DM policies (pairing/allowlist)      │
│  - Channel allowlists                   │
├─────────────────────────────────────────┤
│           ACCESS CONTROL                │
│  - Pairing system                       │
│  - Group policies                       │
│  - Sandbox mode                         │
├─────────────────────────────────────────┤
│           CREDENTIALS                   │
│  - SecretRef for secrets                │
│  - Environment variables                │
├─────────────────────────────────────────┤
│           AUDIT                         │
│  - Security audit command               │
│  - Doctor diagnostics                   │
└─────────────────────────────────────────┘
```

## Quick Start

### Security Audit

Run a security audit to check your configuration:

```bash
# Basic security audit
openclaw security audit

# Deep audit with detailed checks
openclaw security audit --deep

# Auto-fix issues where possible
openclaw security audit --fix
```

### Doctor Diagnostics

Check for common issues and security problems:

```bash
# Run diagnostics
openclaw doctor

# Deep check
openclaw doctor --deep

# Fix issues automatically
openclaw doctor --fix
```

## DM Policies (Direct Message Security)

Control who can send direct messages to your agent:

### Policy Types

| Policy | Behavior | Security Level |
|--------|----------|----------------|
| `pairing` | One-time approval via pairing code | High |
| `allowlist` | Only specified IDs can message | Medium |
| `open` | Anyone can message (use with caution) | Low |
| `disabled` | Block all DMs | Maximum |

### Configuration Examples

**Pairing Mode (Recommended for production):**
```yaml
# ~/.openclaw/config.yaml
channels:
  telegram:
    botToken: "${TELEGRAM_BOT_TOKEN}"
    dmPolicy: pairing  # Requires one-time approval
```

Users must request a pairing code and you must approve it:
```bash
# List pending pairing requests
openclaw pairing list telegram

# Approve a request
openclaw pairing approve telegram ABC123
```

**Allowlist Mode:**
```yaml
channels:
  discord:
    botToken: "${DISCORD_BOT_TOKEN}"
    dmPolicy: allowlist
    allowFrom:
      - "discord:123456789"      # Your Discord ID
      - "discord:987654321"      # Trusted friend
```

**Disabled Mode (channels only):**
```yaml
channels:
  slack:
    botToken: "${SLACK_BOT_TOKEN}"
    dmPolicy: disabled  # No DMs allowed
```

## Group Policies

For channels that support groups (WhatsApp, etc.), control group access:

```yaml
channels:
  whatsapp:
    dmPolicy: allowlist
    allowFrom:
      - "+14155552671"  # Your phone number
    groupPolicy: allowlist  # Options: allowlist, open, disabled
    groupAllowFrom:
      - "group:123456789@g.us"  # Specific group ID
```

## Channel Allowlists

Restrict which channels can access your agent:

```yaml
# ~/.openclaw/config.yaml
security:
  allowed_channels:
    - "telegram:12345678"      # Your private chat
    - "discord:987654321"      # Your private server
    - "slack:U123456789"       # Your Slack user
```

## Secure Credentials with SecretRef

Never store secrets in plain text. Use SecretRef:

```yaml
# ~/.openclaw/config.yaml
channels:
  telegram:
    botToken:
      secretRef: TELEGRAM_BOT_TOKEN  # Reads from OPENCLAW_SECRET_TELEGRAM_BOT_TOKEN
  discord:
    botToken:
      secretRef: DISCORD_BOT_TOKEN

skills:
  entries:
    github:
      config:
        token:
          secretRef: GITHUB_TOKEN
```

**Set environment variables:**
```bash
export OPENCLAW_SECRET_TELEGRAM_BOT_TOKEN="your-token-here"
export OPENCLAW_SECRET_DISCORD_BOT_TOKEN="your-token-here"
export OPENCLAW_SECRET_GITHUB_TOKEN="ghp_xxxxxxxx"
```

## Sandboxing

Control agent sandbox restrictions:

```yaml
# ~/.openclaw/config.yaml
agents:
  defaults:
    sandbox:
      mode: restricted  # Options: full, restricted, off
      network: false    # Disable network access
      filesystem: read-only  # Read-only filesystem
```

| Mode | Description |
|------|-------------|
| `full` | Full sandbox restrictions |
| `restricted` | Partial restrictions (default) |
| `off` | No sandbox (dangerous) |

## Copy-Paste Examples

### Production Security Checklist

```yaml
# ~/.openclaw/config.yaml - Production Hardening

# 1. Use pairing for all DM channels
channels:
  telegram:
    botToken:
      secretRef: TELEGRAM_BOT_TOKEN
    dmPolicy: pairing
  
  discord:
    botToken:
      secretRef: DISCORD_BOT_TOKEN
    dmPolicy: pairing

# 2. Restrict allowed channels
security:
  allowed_channels:
    - "telegram:${TELEGRAM_USER_ID}"
    - "discord:${DISCORD_USER_ID}"

# 3. Enable sandboxing
agents:
  defaults:
    sandbox:
      mode: restricted

# 4. Use SecretRef for all credentials
```

### Pairing Approval Workflow

```bash
# 1. User sends message to bot
# 2. Bot responds with pairing code: "Reply with: pair ABC123"
# 3. User replies: pair ABC123
# 4. Admin checks pending requests:
openclaw pairing list telegram

# 5. Admin approves the request:
openclaw pairing approve telegram ABC123

# 6. User can now message freely
```

### Security Audit Script

```bash
#!/bin/bash
# ~/.openclaw/scripts/security-check.sh

echo "Running OpenClaw security checks..."

# Run security audit
openclaw security audit

# Check for common issues
openclaw doctor

# List all configured channels
openclaw channels list

echo "Security check complete!"
```

## Operational Patterns

### Health Checks

```bash
# Add automated health check
openclaw cron add \
  --name "security-check" \
  --cron "0 */6 * * *" \
  --message "Run openclaw doctor and report any issues" \
  --session isolated
```

### Backup Configuration

```bash
# Backup script for OpenClaw config
cat > ~/.openclaw/scripts/backup.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="$HOME/.openclaw/backups/$DATE"

mkdir -p "$BACKUP_DIR"
cp "$HOME/.openclaw/openclaw.json" "$BACKUP_DIR/"
cp -r "$HOME/.openclaw/skills" "$BACKUP_DIR/"
cp -r "$HOME/.openclaw/memory" "$BACKUP_DIR/"

echo "Backup saved to $BACKUP_DIR"
EOF

chmod +x ~/.openclaw/scripts/backup.sh
```

### Recovery Procedures

```markdown
# ~/.openclaw/RECOVERY.md

## Gateway Won't Start

1. Check logs: `openclaw gateway logs --tail 100`
2. Check port conflicts: `lsof -i :18789`
3. Run diagnostics: `openclaw doctor`
4. Fix issues: `openclaw doctor --fix`
5. Restore from backup: `cp ~/.openclaw/backups/latest/openclaw.json ~/.openclaw/`

## Lost Channel Connection

1. Check token validity
2. Verify bot is still active in channel
3. Re-add channel: `openclaw channels remove --channel <name> --delete`
4. Then: `openclaw channels add --channel <name> --token <new-token>`

## Security Breach Suspected

1. Immediately disable channel: Change dmPolicy to disabled
2. Revoke all tokens
3. Run security audit: `openclaw security audit --deep`
4. Check pairing list: `openclaw pairing list <channel>`
5. Remove unauthorized pairings
```

## Common Mistakes and Troubleshooting

| Problem | Solution |
|---------|----------|
| Unauthorized access | Set dmPolicy to pairing; review allowed_channels |
| Token exposed in config | Use SecretRef instead of plain text |
| Pairing spam | Set dmPolicy to allowlist with specific IDs |
| Gateway security issues | Run `openclaw security audit --fix` |
| Sandbox blocking skills | Check sandbox.mode setting |
| Secrets not loading | Verify OPENCLAW_SECRET_* environment variables |

## Security Best Practices

1. **Always use pairing or allowlist** for DM policies in production
2. **Never commit tokens** - use SecretRef with environment variables
3. **Restrict allowed_channels** to only your personal accounts
4. **Run security audits regularly** - `openclaw security audit`
5. **Keep OpenClaw updated** - `openclaw update`
6. **Backup regularly** - Keep backups of openclaw.json
7. **Review skills** before installing - `openclaw skills check`
8. **Use sandboxing** - Don't disable sandbox in production

## Related modules and next step

- Previous: [08-workflows](../08-workflows/)
- Next: [10-cli-and-reference](../10-cli-and-reference/)
- Reference: [Security documentation](https://docs.openclaw.ai/gateway/security)

---

**Time to complete:** 45 minutes  
**Prerequisites:** [08-workflows](../08-workflows/)  
**Outcome:** Secure, production-ready OpenClaw deployment with proper access controls
