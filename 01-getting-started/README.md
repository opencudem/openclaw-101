# 01 - Getting Started

> Install OpenClaw and get your first response in under 30 minutes.

## What this module solves

This module takes you from zero to a working OpenClaw Gateway with your first connected channel. By the end, you'll be chatting with your AI assistant.

## When to use this module

- You've never installed OpenClaw before
- You want a clean, verified installation
- You need to understand the basic architecture

## Quick Start

### Prerequisites

- Node.js 18+ installed
- npm or yarn
- One chat app account (Telegram, Discord, WhatsApp, or Slack)

### Installation

```bash
# Official install script (recommended)
curl -fsSL https://openclaw.ai/install.sh | bash

# Alternative: npm install
npm install -g openclaw@latest

# Verify installation
openclaw --version
```

### Onboarding and Gateway Setup

```bash
# Interactive onboarding with daemon installation
openclaw onboard --install-daemon

# Or quick setup flow (minimal prompts)
openclaw onboard --flow quickstart
```

**Onboarding flags:**
- `--install-daemon`: Install system service (launchd/systemd/schtasks)
- `--flow quickstart|manual`: Choose prompt level
- `--mode local|remote`: Gateway location
- `--non-interactive`: For automation/scripts
- `--accept-risk`: Acknowledge security considerations

### Run the Gateway

```bash
# Run gateway in foreground (development)
openclaw gateway

# Or use alias
openclaw gateway run

# Check status
openclaw gateway status
openclaw gateway status --require-rpc
```

### Connect Your First Channel

**Option A: Telegram (Easiest)**

```bash
# 1. Create a bot with @BotFather on Telegram
# 2. Get your bot token
# 3. Add the channel
openclaw channels add --channel telegram --token YOUR_BOT_TOKEN
```

**Option B: Discord**

```bash
# 1. Create a Discord application at https://discord.com/developers/applications
# 2. Enable Bot scope, get token
# 3. Add the channel
openclaw channels add --channel discord --token YOUR_BOT_TOKEN
```

### Verify Everything Works

Send a message to your bot:

```
hello
```

You should get a response within seconds.

## How it works under the hood

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Your Chat App  │◄───►│   Gateway    │◄───►│  AI Assistant   │
│ (Telegram/etc)  │     │  (openclaw)  │     │  (Claude/etc)   │
└─────────────────┘     └──────────────┘     └─────────────────┘
```

The Gateway is the middleman: it receives messages from your chat apps, forwards them to the AI, and sends responses back.

## Copy-paste examples

### Check Gateway health

```bash
openclaw gateway status
openclaw gateway logs --tail 50
```

### List connected channels

```bash
openclaw channels list
openclaw channels status
```

## Common mistakes and troubleshooting

| Problem | Solution |
|---------|----------|
| "command not found: openclaw" | Make sure npm global bin is in your PATH |
| Gateway won't start | Check port 18789 isn't in use: `lsof -i :18789` |
| Bot doesn't respond | Verify token is correct; check gateway logs |
| Messages delayed | Check your internet connection and API credits |

## Advanced patterns

### Run Gateway on a different port

```bash
openclaw gateway --port 8080
```

### Environment variables

Create a `.env` file in `~/.openclaw/.env`:

```env
OPENCLAW_PORT=18789
OPENCLAW_LOG_LEVEL=info
OPENCLAW_PROVIDER=anthropic
```

## Related modules and next step

- Next: [02-channels](../02-channels/) — Add more channels
- Reference: [Gateway CLI](../10-cli-and-reference/gateway.md)
- Troubleshooting: [Gateway won't start](../TROUBLESHOOTING.md#gateway-wont-start)

---

**Time to complete:** 30 minutes  
**Prerequisites:** None  
**Outcome:** Working OpenClaw installation with first channel
