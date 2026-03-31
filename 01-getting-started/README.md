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
# Install OpenClaw globally
npm install -g openclaw

# Verify installation
openclaw --version

# Initialize configuration
openclaw init
```

### Start the Gateway

```bash
# Start the Gateway daemon
openclaw gateway start

# Check status
openclaw gateway status
```

### Connect Your First Channel

**Option A: Telegram (Easiest)**

```bash
# 1. Create a bot with @BotFather on Telegram
# 2. Get your bot token
# 3. Add the channel
openclaw channel add telegram --token YOUR_BOT_TOKEN
```

**Option B: Discord**

```bash
# 1. Create a Discord application at https://discord.com/developers/applications
# 2. Enable Bot scope, get token
# 3. Add the channel
openclaw channel add discord --token YOUR_BOT_TOKEN
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
openclaw channel list
```

### Pause/resume a channel

```bash
openclaw channel pause telegram
openclaw channel resume telegram
```

## Common mistakes and troubleshooting

| Problem | Solution |
|---------|----------|
| "command not found: openclaw" | Make sure npm global bin is in your PATH |
| Gateway won't start | Check port 3000 isn't in use: `lsof -i :3000` |
| Bot doesn't respond | Verify token is correct; check gateway logs |
| Messages delayed | Check your internet connection and API credits |

## Advanced patterns

### Run Gateway on a different port

```bash
openclaw gateway start --port 8080
```

### Environment variables

Create a `.env` file:

```env
OPENCLAW_PORT=3000
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
