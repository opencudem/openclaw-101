# iMessage via BlueBubbles — Complete Setup Guide

> Connect OpenClaw to iMessage using BlueBubbles server on macOS. Send and receive iMessages, reactions, attachments, and group chat messages through your OpenClaw agent.

---

## Overview

**BlueBubbles** is an open-source macOS server that exposes iMessage over a REST API. OpenClaw connects to it for:

- ✅ Send/receive iMessages (DMs and groups)
- ✅ Tapback reactions (👍❤️😂 etc.)
- ✅ Message editing and unsending (macOS 13+)
- ✅ Attachments and voice memos
- ✅ Group management (add/remove participants)
- ✅ Read receipts and typing indicators

**Status**: Production-ready | **Requires**: Physical Mac running macOS 12+

---

## ⚠️ Critical Requirements

**A Mac is NOT optional.** iMessage is Apple-only infrastructure. You need:

- Mac running macOS Sequoia (15) or later (macOS 26 Tahoe supported with caveats)
- Messages.app signed into an Apple ID
- Mac must stay powered on and connected to internet
- No Docker workaround exists — BlueBubbles runs directly on macOS

---

## Step 1: Install BlueBubbles Server

### 1.1 Download and Install

1. Go to <https://bluebubbles.app/install>
2. Download the latest `.dmg` for macOS
3. Open the DMG and drag BlueBubbles to Applications
4. Launch BlueBubbles from Applications

### 1.2 Grant macOS Permissions

On first launch, macOS will request several permissions. **Grant all of them**:

| Permission | Why It’s Needed |
|------------|-----------------|
| **Automation** | Control Messages.app to send/receive texts |
| **Full Disk Access** | Read the Messages database (`chat.db`) |
| **Accessibility** | Interact with Messages UI for advanced features |

Go to **System Settings → Privacy & Security** to verify all permissions are granted.

### 1.3 Complete Setup Wizard

The BlueBubbles setup wizard will guide you through:

1. **iMessage Account** — Use the default (your signed-in Apple ID)
2. **Server Password** — Set a strong password (you'll need this for OpenClaw)
3. **Web API** — Enable this (required for OpenClaw)
4. **Private API** — Enable for reactions, edit, unsend features

### 1.4 Verify Server is Running

Open Terminal on your Mac and test:

```bash
curl http://localhost:1234/api/v1/ping
```

Expected response:
```json
{"status":"ok","timestamp":1715530507}
```

If you get a response, BlueBubbles is running. Note the port (default: 1234).

---

## Step 2: Network Access

OpenClaw needs to reach your Mac. Choose your setup:

### Option A: Same Machine (OpenClaw + BlueBubbles on same Mac)

Use `localhost` — simplest setup.

### Option B: Different Machine (OpenClaw on VPS, BlueBubbles on Mac)

Your Mac needs to be reachable from your OpenClaw host:

| Method | Setup | Best For |
|--------|-------|----------|
| **Tailscale** | Install Tailscale on both, use Tailscale IP | Most secure, recommended |
| **ngrok** | `ngrok http 1234` on Mac, use ngrok URL | Quick testing |
| **Public IP** | Port forward 1234 on router | Home servers with static IP |

**Recommended**: Tailscale. It creates a secure mesh VPN between your devices.

---

## Step 3: Configure OpenClaw

### 3.1 Interactive Setup (Recommended)

Run the onboarding wizard:

```bash
openclaw onboard
```

Select **BlueBubbles** when prompted. You'll need:
- Server URL (e.g., `http://192.168.1.100:1234` or `http://mac-mini.tailnet.ts.net:1234`)
- Password (from BlueBubbles settings)

### 3.2 Manual Configuration

Edit your `~/.openclaw/openclaw.json` or `config.json5`:

```json5
{
  channels: {
    bluebubbles: {
      enabled: true,
      serverUrl: "http://192.168.1.100:1234",  // Your Mac's IP or Tailscale address
      password: "your-strong-password-here",  // From BlueBubbles settings
      webhookPath: "/bluebubbles-webhook",
      dmPolicy: "pairing",  // Require approval for new DMs
      sendReadReceipts: true,
      actions: {
        reactions: true,     // Tapbacks (👍❤️😂)
        edit: true,          // Edit sent messages (macOS 13+)
        unsend: true,        // Unsend messages (macOS 13+)
        reply: true,         // Reply threading
        sendWithEffect: true, // iMessage effects (slam, loud, gentle)
        renameGroup: true,   // Rename group chats
        setGroupIcon: true,  // Set group photo
        addParticipant: true,
        removeParticipant: true,
        leaveGroup: true,
        sendAttachment: true, // Send files/images
      },
    },
  },
}
```

### 3.3 Add Channel via CLI

```bash
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password "your-password"
```

---

## Step 4: Webhook Configuration

Incoming iMessages arrive via webhooks. You must tell BlueBubbles where to send them.

### 4.1 Get Your Gateway URL

Your OpenClaw gateway needs a public or reachable URL. Format:

```
https://your-gateway-host:3000/bluebubbles-webhook?password=YOUR_WEBHOOK_PASSWORD
```

**Security best practice**: Use a different password for webhooks than the API password.

### 4.2 Configure BlueBubbles Webhook

In BlueBubbles Server settings:

1. Go to **Settings → Webhook**
2. Enable **Webhook**
3. Set **Webhook URL** to your gateway URL (above)
4. Add the password as query param or header

### 4.3 Test Webhook

Send a test message from another device. Check OpenClaw logs:

```bash
openclaw logs --follow
```

You should see incoming message events.

---

## Step 5: Pairing and Access Control

### 5.1 DM Pairing (Default: Enabled)

By default, unknown senders must be approved. When someone new messages you:

```bash
# List pending pairings
openclaw pairing list bluebubbles

# Approve a sender
openclaw pairing approve bluebubbles <CODE>

# Deny a sender
openclaw pairing deny bluebubbles <CODE>
```

Pairing codes expire after **1 hour**.

### 5.2 Allowlist (Skip Pairing)

To auto-approve specific contacts:

```json5
{
  channels: {
    bluebubbles: {
      dmPolicy: "allowlist",
      allowFrom: [
        "+15555550123",        // Phone number (E.164 format)
        "friend@example.com",    // Email
        "chat_id:123",          // Specific chat ID
      ],
    },
  },
}
```

### 5.3 Group Chat Settings

```json5
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",  // or "open" for any group
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },  // Default: must @mention bot
        "iMessage;-;chat123": { requireMention: false },  // Override
      },
    },
  },
}
```

---

## Step 6: Keeping Messages.app Alive

On headless Macs or VMs, Messages.app can go "idle" and stop receiving events.

### Solution: Poke Messages Every 5 Minutes

Create `~/Scripts/poke-messages.scpt`:

```applescript
try
  tell application "Messages"
    if not running then
      launch
    end if
    set _chatCount to (count of chats)
  end tell
on error
  -- Ignore transient failures
end try
```

Create `~/Library/LaunchAgents/com.user.poke-messages.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.poke-messages</string>
    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>-lc</string>
      <string>/usr/bin/osascript "$HOME/Scripts/poke-messages.scpt"</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>StartInterval</key>
    <integer>300</integer>
  </dict>
</plist>
```

Load it:

```bash
launchctl load ~/Library/LaunchAgents/com.user.poke-messages.plist
```

---

## Troubleshooting

### "No messages coming through"

| Check | Command |
|-------|---------|
| BlueBubbles running? | `curl http://localhost:1234/api/v1/ping` |
| Gateway reachable? | Check webhook URL is correct |
| Pairing pending? | `openclaw pairing list bluebubbles` |
| Firewall blocking? | Verify port 1234 is open |

### "Reactions not working"

Enable Private API in BlueBubbles settings. Reactions require it.

### "Edit/unsend broken on macOS Tahoe (26)"

Known issue — private API changes in Tahoe broke edit. Disable it:

```json5
{
  channels: {
    bluebubbles: {
      actions: {
        edit: false,
      },
    },
  },
}
```

### "Messages.app goes idle"

Use the LaunchAgent solution above (Step 6).

### Full Diagnostic

```bash
openclaw status --deep
openclaw channels status --probe
```

---

## Advanced: ACP Bindings

Turn any iMessage chat into a persistent ACP workspace:

```bash
# Inside a BlueBubbles DM or group
/acp spawn codex --bind here
```

Future messages route to the spawned ACP session.

Or configure persistent binding in `openclaw.json`:

```json5
{
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "bluebubbles",
        peer: { kind: "dm", id: "+15555550123" },
      },
    },
  ],
}
```

---

## Configuration Reference

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enabled` | boolean | false | Enable channel |
| `serverUrl` | string | "" | BlueBubbles REST API URL |
| `password` | string | "" | API password |
| `webhookPath` | string | "/bluebubbles-webhook" | Webhook endpoint |
| `dmPolicy` | string | "pairing" | "pairing", "allowlist", "open", "disabled" |
| `groupPolicy` | string | "allowlist" | "allowlist", "open", "disabled" |
| `sendReadReceipts` | boolean | true | Send read receipts |
| `blockStreaming` | boolean | false | Stream responses in blocks |
| `mediaMaxMb` | number | 8 | Max attachment size (MB) |
| `textChunkLimit` | number | 4000 | Max characters per message |

---

## Next Steps

- [Pairing & Access Control](../README.md#access-control)
- [Group Chat Behavior](../README.md#groups)
- [Channel Routing](../README.md#routing)
- [Security Best Practices](../../09-security-and-ops/README.md)

---

*Last updated: April 4, 2026 — OpenClaw v2026.4.2*
