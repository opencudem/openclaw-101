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
| **iMessage** | Apple ecosystem | 🟡 Medium |
| **Google Chat** | Google Workspace users | 🟡 Medium |

## Quick Start by Channel

### Telegram

```bash
# 1. Message @BotFather on Telegram
# 2. Send /newbot and follow instructions
# 3. Copy the bot token (looks like 123456789:ABCdefGHIjklMNOpqrsTUVwxyz)

# 4. Add to OpenClaw
openclaw channel add telegram --token YOUR_BOT_TOKEN

# 5. Start chatting with your bot
```

**Pro tip:** Create bot commands with `/setcommands` in BotFather for quick access.

### Discord

```bash
# 1. Go to https://discord.com/developers/applications
# 2. Create New Application → Bot → Enable Bot
# 3. Copy token and enable Message Content Intent
# 4. Generate OAuth2 URL with bot scope, send messages permission
# 5. Invite bot to your server
# 6. Add to OpenClaw
openclaw channel add discord --token YOUR_BOT_TOKEN
```

**Pro tip:** Create a private channel just for you and the bot.

### WhatsApp

```bash
# WhatsApp requires a phone number
# Options:
# 1. Use a dedicated number (recommended)
# 2. Use WhatsApp Business with a second device

openclaw channel add whatsapp --type baileys
# Follow the QR code prompt to pair
```

⚠️ **Important:** WhatsApp has strict rate limits. Don't use your primary number.

### Slack

```bash
# 1. Go to https://api.slack.com/apps
# 2. Create New App → From scratch
# 3. Add Bot Token Scopes: chat:write, im:history, im:read
# 4. Install to workspace, copy Bot User OAuth Token
# 5. Add to OpenClaw
openclaw channel add slack --token xoxb-YOUR-TOKEN
```

## Multi-Channel Setup

You can connect multiple channels at once:

```bash
# Add all your channels
openclaw channel add telegram --token TELEGRAM_TOKEN
openclaw channel add discord --token DISCORD_TOKEN
openclaw channel add slack --token SLACK_TOKEN

# List all channels
openclaw channel list
```

Messages sent to any channel route to the same agent. The agent knows which channel you're using.

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
    - telegram:12345678  # Your private chat
    - discord:98765432 # Your private server
```

## Copy-paste examples

### Get channel ID

```bash
# Send a message from the channel
# Check logs for the channel identifier
openclaw gateway logs | grep "channel"
```

### Pause/resume specific channels

```bash
openclaw channel pause whatsapp
openclaw channel resume whatsapp
```

### Remove a channel

```bash
openclaw channel remove telegram
```

## Common mistakes and troubleshooting

| Problem | Solution |
|---------|----------|
| Telegram bot not responding | Check if you messaged the right bot; verify token |
| Discord bot offline | Check Message Content Intent is enabled in Bot settings |
| WhatsApp QR code fails | Use dedicated number; clear previous pairings |
| Slack bot can't DM | Bot needs to be invited to the channel first |

## Advanced patterns

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

## Related modules and next step

- Previous: [01-getting-started](../01-getting-started/)
- Next: [03-memory](../03-memory/) — Personalize with memory
- Related: [Channel troubleshooting](../TROUBLESHOOTING.md#channels)

---

**Time to complete:** 45 minutes  
**Prerequisites:** [01-getting-started](../01-getting-started/)  
**Outcome:** Multiple channels connected, messaging from any platform
