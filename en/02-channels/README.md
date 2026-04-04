# 02 - Channels

> Connect OpenClaw to where you already chat: WhatsApp, Telegram, Discord, and Slack.

## What this module solves

OpenClaw's defining feature is meeting you where you are. This module shows you how to connect multiple messaging platforms so you can talk to your assistant from any device.

## When to use this module

- You want to use OpenClaw from your phone
- Your team uses multiple chat platforms
- You want channel-specific behaviors

## Supported Channels

| Channel | Best For | Difficulty |
|---------|----------|------------|
| **Telegram** | Quick start, mobile access | 🟢 Easy |
| **Discord** | Team collaboration, communities | 🟢 Easy |
| **WhatsApp** | Personal use, existing contacts | 🟡 Medium |
| **Slack** | Workplace integration | 🟡 Medium |
| **iMessage** | Apple ecosystem | 🟡 Medium | [BlueBubbles Guide →](./bluebubbles-guide/) |
| **Signal** | Privacy-focused users | 🟡 Medium |

## Channel Commands Overview

All channel management uses the plural `channels` command:

```bash
# List connected channels
openclaw channels list

# Check channel capabilities
openclaw channels capabilities

# Add a channel (examples below for each platform)
openclaw channels add --channel <name> --token <token>

# Remove a channel
openclaw channels remove --channel <name> --delete

# Interactive login (for QR pairing)
openclaw channels login --channel <name>

# Logout
openclaw channels logout --channel <name>

# Resolve names to IDs
openclaw channels resolve --channel slack "#general" "@jane"
```

## Quick Start by Channel

### Telegram

```bash
# 1. Message @BotFather on Telegram
# 2. Send /newbot and follow instructions
# 3. Copy the bot token (looks like 123456789:ABCdefGHIjklMNOpqrsTUVwxyz)

# 4. Add to OpenClaw
openclaw channels add --channel telegram --token YOUR_BOT_TOKEN

# 5. Configure DM policy (optional)
# Edit ~/.openclaw/config.yaml or use environment variables
# 6. Start chatting with your bot
```

**DM Policies for Telegram:**

Telegram supports direct messages. Configure who can message your agent:

```yaml
channels:
  telegram:
    botToken: "${TELEGRAM_BOT_TOKEN}"
    dmPolicy: pairing  # Options: pairing, allowlist, open, disabled
```

**DM Policy Options:**
- `pairing` (default): One-time pairing code approval required
- `allowlist`: Only senders in `allowFrom` list can message
- `open`: Allow all inbound (requires `allowFrom: ["*"]`)
- `disabled`: Block all direct messages

**Pairing Commands:**

When using `dmPolicy: pairing`, users must pair before messaging:

```bash
# List pending pairing requests
openclaw pairing list telegram

# Approve a pairing request
openclaw pairing approve telegram <CODE>

# Pairing codes expire after 1 hour
```

**Pro tip:** Create bot commands with `/setcommands` in BotFather for quick access.

---

### Discord

```bash
# 1. Go to https://discord.com/developers/applications
# 2. Create New Application → Bot → Add Bot
# 3. Copy the Bot Token
# 4. Enable Message Content Intent (required for bot to read messages)
# 5. Generate OAuth2 URL with bot scope and send messages permission
# 6. Invite bot to your server
# 7. Add to OpenClaw
openclaw channels add --channel discord --token YOUR_BOT_TOKEN
```

**Required Discord Setup:**

1. **Privileged Intents** (required):
   - Enable `MESSAGE CONTENT INTENT` in Bot → Privileged Gateway Intents
   - Without this, the bot cannot read message content

2. **OAuth2 Scopes** for invite URL:
   - `bot` scope
   - `Send Messages` permission
   - `Read Message History` permission

**DM Policies for Discord:**

Discord supports DMs. Configure access control:

```yaml
channels:
  discord:
    botToken: "${DISCORD_BOT_TOKEN}"
    dmPolicy: allowlist
    allowFrom:
      - "discord:123456789"  # Your Discord user ID
```

**Pro tip:** Create a private channel just for you and the bot, or use DMs for secure communication.

---

### WhatsApp

WhatsApp requires QR code pairing (interactive process):

```bash
# 1. Add WhatsApp channel (no token needed initially)
openclaw channels add --channel whatsapp

# 2. Login to initiate QR pairing
openclaw channels login --channel whatsapp
# A QR code will appear - scan with WhatsApp app

# 3. Once paired, your session is saved
```

**WhatsApp Configuration:**

```yaml
channels:
  whatsapp:
    allowFrom:
      - "+14155552671"  # Your phone number in E.164 format
    groupPolicy: allowlist  # Options: allowlist, open, disabled
    groupAllowFrom:
      - "group:123456789"  # Specific group IDs
```

**Important WhatsApp Notes:**

- **E.164 Format**: Phone numbers must include `+` and country code (e.g., `+14155552671`)
- **QR Pairing**: Run `openclaw channels login --channel whatsapp` to re-pair if session expires
- **Rate Limits**: WhatsApp has strict rate limits. Don't spam.
- **Dedicated Number**: Use a secondary number, not your primary

**WhatsApp Group Support:**

```yaml
# Allow specific groups
channels:
  whatsapp:
    groupPolicy: allowlist
    groupAllowFrom:
      - "group:123456789@g.us"
```

---

### Slack

Slack supports two modes: **Socket Mode** (default, recommended) and **HTTP Mode**.

```bash
# 1. Go to https://api.slack.com/apps
# 2. Create New App → From scratch
# 3. Add features: Bots, Permissions
# 4. In OAuth & Permissions, add Bot Token Scopes:
#    - chat:write
#    - im:history
#    - im:read
#    - channels:history (for channel access)
# 5. Install to workspace, copy Bot User OAuth Token (starts with xoxb-)
# 6. For Socket Mode, also get App-Level Token (starts with xapp-)

# 7. Add to OpenClaw
openclaw channels add --channel slack --token xoxb-YOUR-BOT-TOKEN
```

**Slack Configuration:**

```yaml
channels:
  slack:
    botToken: "${SLACK_BOT_TOKEN}"
    appToken: "${SLACK_APP_TOKEN}"  # Required for Socket Mode
    mode: socket  # or 'http' for webhook mode
    dmPolicy: allowlist
    allowFrom:
      - "slack:U123456789"  # Your Slack user ID
```

**Required Bot Token Scopes:**
- `chat:write` - Send messages
- `im:history` - Access DM history
- `im:read` - Read DMs
- `channels:history` - Access channel history (optional)

**Socket Mode vs HTTP Mode:**

| Mode | Best For | Token Required |
|------|----------|----------------|
| **Socket Mode** | Local development, behind firewall | Bot Token + App Token |
| **HTTP Mode** | Servers with public URL | Bot Token only |

**Pro tip:** The bot needs to be invited to channels before it can see messages. Use `/invite @YourBot` in each channel.

---

## Multi-Channel Setup

You can connect multiple channels at once:

```bash
# Add all your channels
openclaw channels add --channel telegram --token TELEGRAM_TOKEN
openclaw channels add --channel discord --token DISCORD_TOKEN
openclaw channels add --channel slack --token xoxb-SLACK_TOKEN

# List all connected channels
openclaw channels list

# Check what each channel supports
openclaw channels capabilities
```

Messages sent to any channel route to the same agent. The agent knows which channel you're using.

---

## DM Policies (Direct Message Security)

Control who can send direct messages to your agent:

### Policy Types

| Policy | Behavior | Use Case |
|--------|----------|----------|
| **pairing** | One-time approval via pairing code | Secure, controlled access |
| **allowlist** | Only specified IDs can message | Known contacts only |
| **open** | Anyone can message (use with caution) | Public bots |
| **disabled** | Block all DMs | Channels-only mode |

### Configuration Examples

**Pairing Mode (most secure):**
```yaml
channels:
  telegram:
    botToken: "${TELEGRAM_BOT_TOKEN}"
    dmPolicy: pairing
```

User must request pairing:
```
User: Can I talk to you?
Bot: Pairing required. Reply with: pair ABC123
User: pair ABC123
# Admin runs: openclaw pairing approve telegram ABC123
```

**Allowlist Mode:**
```yaml
channels:
  discord:
    botToken: "${DISCORD_BOT_TOKEN}"
    dmPolicy: allowlist
    allowFrom:
      - "discord:123456789"      # Your Discord ID
      - "discord:987654321"      # Friend's Discord ID
```

**Open Mode (public bot):**
```yaml
channels:
  telegram:
    botToken: "${TELEGRAM_BOT_TOKEN}"
    dmPolicy: open
    allowFrom: ["*"]  # Required when using 'open'
```

---

## Pairing Commands

For channels using `dmPolicy: pairing`:

```bash
# List all pending pairing requests
openclaw pairing list

# List requests for a specific channel
openclaw pairing list telegram

# Approve a pairing request
openclaw pairing approve telegram ABC123

# Approve for Discord
openclaw pairing approve discord XYZ789
```

**Pairing Flow:**
1. User messages bot and requests to pair
2. Bot provides a pairing code (e.g., `ABC123`)
3. Admin runs `openclaw pairing approve <channel> <code>`
4. User can now message the bot freely
5. Pairing codes expire after 1 hour

---

## Channel-Specific Behaviors

### Different responses per channel

Configure in your OpenClaw settings:

```yaml
# ~/.openclaw/config.yaml
channels:
  telegram:
    personality: "casual, emoji-friendly"
  discord:
    personality: "technical, markdown-friendly"
  slack:
    personality: "professional, concise"
```

### Restricting channels

Limit which channels can execute sensitive commands:

```yaml
security:
  allowed_channels:
    - "telegram:12345678"   # Your private chat
    - "discord:98765432"    # Your private server
```

---

## Copy-paste Examples

### Get channel ID

```bash
# Send a message from the channel
# Check logs for the channel identifier
openclaw gateway logs | grep "channel"

# Or use the resolve command
openclaw channels resolve --channel slack "#general"
```

### Remove a channel

```bash
# Remove Telegram
openclaw channels remove --channel telegram --delete

# Remove Discord
openclaw channels remove --channel discord --delete

# The --delete flag removes the channel from config
```

### Check channel status

```bash
# List all channels and their status
openclaw channels list

# Check what a channel supports
openclaw channels capabilities --channel telegram
```

---

## Common Mistakes and Troubleshooting

| Problem | Solution |
|---------|----------|
| Telegram bot not responding | Check if you messaged the right bot; verify token; check `openclaw gateway logs` |
| Discord bot offline | Check Message Content Intent is enabled in Bot settings |
| WhatsApp QR code fails | Use dedicated number; clear previous pairings with `openclaw channels logout --channel whatsapp` |
| Slack bot can't DM | Bot needs to be invited to the channel first; check scopes include `im:read` |
| "Invalid command: channel" | Use `channels` (plural), not `channel` |
| Pairing code expired | Codes expire after 1 hour; user must request a new one |
| DMs blocked | Check `dmPolicy` setting and `allowFrom` list |

---

## Advanced Patterns

### Webhook mode (for servers)

Instead of polling, receive webhooks:

```yaml
channels:
  telegram:
    mode: webhook
    webhook_url: https://your-server.com/webhook/telegram
```

### Channel routing rules

Route different message types to different channels:

```yaml
routing:
  alerts:
    channels: [slack, discord]
  personal:
    channels: [telegram]
```

### Using SecretRef for tokens

Store tokens securely instead of in config files:

```yaml
channels:
  discord:
    botToken:
      secretRef: discord-bot-token  # References OPENCLAW_SECRET_discord_bot_token env var
```

---

## Related Modules and Next Steps

- Previous: [01-getting-started](../01-getting-started/)
- Next: [03-memory](../03-memory/) — Personalize with memory
- Related: [Channel troubleshooting](../TROUBLESHOOTING.md#channels)

---

**Time to complete:** 45 minutes  
**Prerequisites:** [01-getting-started](../01-getting-started/)  
**Outcome:** Multiple channels connected, messaging from any platform
