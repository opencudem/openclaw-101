# 12 — Heartbeat Configuration

> Configure periodic agent check-ins for proactive automation and maintenance workflows.

## What this module solves

Heartbeat runs **periodic agent turns** so your agent can surface important information without you having to ask. Unlike cron jobs (which run in the background), heartbeat creates a **main session turn** where the agent can:

- Check for urgent follow-ups (emails, calendar, notifications)
- Review and update documentation
- Run maintenance workflows (like DocLoop)
- Surface alerts that need human attention

**Key difference from cron:**
- **Cron** = Background task execution, creates task records, cannot trigger the agent
- **Heartbeat** = Main session turn, direct agent interaction, no task records

## When to use this module

- You want **proactive check-ins** without manual prompting
- You need **periodic maintenance** (documentation, cleanup, monitoring)
- You want **batch multiple checks** into one turn (inbox + calendar + weather)
- You need **conversational context** from recent messages

**Use cron instead when:**
- Exact timing matters ("9:00 AM sharp every Monday")
- Task needs isolation from main session history
- One-shot reminders ("remind me in 20 minutes")
- Output should deliver directly without main session involvement

## Prerequisites

**OpenClaw Version:** v0.31+ (for `heartbeat` configuration)  
**Version Introduced:** v2026.4.x series added enhanced `lightContext` and `isolatedSession` options  
**Required:** Working Gateway with at least one channel configured

> 📸 **[SCREENSHOT NEEDED: openclaw.json heartbeat config in VS Code editor with syntax highlighting]**

## Quick Start

### Step 1: Create HEARTBEAT.md Checklist

Create a small checklist in your workspace:

```bash
cat > ~/.openclaw/workspace/HEARTBEAT.md << 'EOF'
# Heartbeat Checklist

## DocLoop Maintenance
- [ ] Check for new OpenClaw releases
- [ ] Review AUDIT_REPORT.md for gaps
- [ ] Update documentation if needed

## System Health
- [ ] Quick scan: urgent inboxes or notifications
- [ ] Check gateway status

## Response Rules
- If nothing needs attention → reply HEARTBEAT_OK
- If work found → do work and report results
- If blocked → note what's needed and ask human
EOF
```

Keep it **tiny** to avoid token burn. If the file is empty (only headers/blank lines), heartbeat skips to save API calls.

### Step 2: Add Heartbeat Configuration

Edit `~/.openclaw/openclaw.json`:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",              // Run every 30 minutes
        target: "discord",       // Deliver to Discord
        accountId: "default",    // Use this Discord account
        to: "YOUR_CHANNEL_ID",   // Specific channel (optional)
        
        // Cost optimization (recommended for maintenance workflows):
        lightContext: true,        // Only load HEARTBEAT.md
        isolatedSession: true,     // Fresh session each time
        
        prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
      },
    },
  },
}
```

### Step 3: Restart Gateway

```bash
openclaw gateway restart
```

### Step 4: Verify

Wait 30 minutes or trigger manually:

```bash
openclaw system event --text "Test heartbeat" --mode now
```

## Configuration Deep Dive

### `every` — Interval

```json5
heartbeat: {
  every: "30m",    // 30 minutes (default)
  // every: "1h",  // 1 hour
  // every: "0m",  // Disable heartbeat
}
```

**Default:** `30m` (or `1h` for Anthropic OAuth/setup-token mode)

### `target` — Delivery Destination

| Value | Behavior |
|-------|----------|
| `"none"` (default) | Run but don't deliver externally |
| `"last"` | Deliver to last used external channel |
| `"discord"` | Deliver to Discord |
| `"telegram"` | Deliver to Telegram |
| `"whatsapp"` | Deliver to WhatsApp |

**With account and channel:**

```json5
heartbeat: {
  target: "discord",
  accountId: "docloop",           // Multi-account channels
  to: "1488561209300095230",      // Channel ID (Discord)
  // to: "+15551234567",          // Phone (WhatsApp)
  // to: "12345678:topic:42",     // Telegram thread
}
```

### `lightContext` — Minimal Bootstrap

| Value | Bootstrap Files | Token Cost | Use Case |
|-------|-----------------|------------|----------|
| `false` (default) | All files: `AGENTS.md`, `SOUL.md`, `USER.md`, `TOOLS.md`, `HEARTBEAT.md` | ~10-50K tokens | Full persona context |
| `true` | **Only `HEARTBEAT.md`** | ~1-5K tokens | Checklist-only workflows |

**Recommended for:**
- DocLoop maintenance (only needs checklist)
- Inbox/calendar checks (doesn't need personality)
- System monitoring (technical tasks)

### `isolatedSession` — Fresh Session Per Run

| Value | Session Behavior | Token Cost |
|-------|------------------|------------|
| `false` (default) | Continues main session with full history | ~50-100K+ tokens |
| `true` | **Fresh session, no prior context** | ~2-5K tokens |

**Recommended for:**
- Maintenance workflows (each run is independent)
- Testing/validation (no stale context)
- Cost-sensitive setups (~90% token savings)

**⚠️ Trade-off:** Cannot access prior work or conversation history.

### `activeHours` — Time Windows

Restrict heartbeats to business hours:

```json5
heartbeat: {
  every: "30m",
  activeHours: {
    start: "09:00",              // 9 AM
    end: "22:00",                // 10 PM
    timezone: "Asia/Ho_Chi_Minh", // Your timezone (or "user" for default)
  },
}
```

Outside this window, heartbeats are skipped.

### `includeReasoning` — Transparency

```json5
heartbeat: {
  includeReasoning: true,  // Also deliver Reasoning: message
}
```

Sends separate `Reasoning:` message showing why the agent decided to alert you. Useful for debugging but can leak internal details.

### `prompt` — Custom Instructions

Override the default prompt:

```json5
heartbeat: {
  prompt: "Check GitHub releases for new OpenClaw versions. If v2026.4.3+ released, start DocLoop Phase 1. Otherwise reply HEARTBEAT_OK.",
}
```

## Response Contract

### Agent Rules

| Condition | Response |
|-----------|----------|
| Nothing needs attention | `HEARTBEAT_OK` (only) |
| Work found | Report results, do NOT include HEARTBEAT_OK |
| Blocked/needs human | Alert with specific ask |

### System Behavior

- `HEARTBEAT_OK` at **start or end** → stripped, message dropped if ≤ 300 chars remaining
- `HEARTBEAT_OK` in **middle** → not treated specially
- Full alert text → delivered to `target` channel

## Per-Agent Heartbeats

If any `agents.list[]` has a `heartbeat` block, **only those agents** run heartbeats:

```json5
{
  agents: {
    defaults: {
      heartbeat: { every: "30m", target: "last" },  // Shared defaults
    },
    list: [
      { id: "main" },                              // No heartbeat
      {
        id: "docloop",
        heartbeat: {                               // Runs heartbeats
          every: "1h",
          target: "discord",
          accountId: "docloop",
        }
      },
    ],
  },
}
```

The per-agent block merges on top of defaults.

## Cost Optimization

| Strategy | Config | Savings |
|----------|--------|---------|
| Use `lightContext: true` | Only `HEARTBEAT.md` | ~80% |
| Use `isolatedSession: true` | Fresh session | ~90% |
| Combine both | `lightContext + isolatedSession` | ~95% |
| Cheaper model | `model: "ollama/llama3.2:1b"` | ~99% |

**Example cost-optimized config:**

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "discord",
        lightContext: true,
        isolatedSession: true,
        model: "custom-api-fireworks-ai/accounts/fireworks/routers/kimi-k2p5-turbo",
      },
    },
  },
}
```

## Common Mistakes

| Mistake | Why It's Bad | Solution |
|---------|--------------|----------|
| No `heartbeat` config | Heartbeat never runs | Add `agents.defaults.heartbeat` or per-agent config |
| `target: "none"` and expecting messages | Silent operation | Use `target: "last"` or specific channel |
| `lightContext: true` but needing `SOUL.md` | Agent acts generic | Use `lightContext: false` for persona context |
| `isolatedSession: true` and needing history | Can't access prior work | Use `isolatedSession: false` or store in files |
| Empty `HEARTBEAT.md` | Heartbeat skips | Add actual checklist items |

## Troubleshooting

### "I don't see any heartbeat"

**Check 1:** Is heartbeat configured?
```bash
grep -A 10 "heartbeat" ~/.openclaw/openclaw.json
```

**Check 2:** Is gateway running?
```bash
openclaw gateway status
```

**Check 3:** Has it been 30 minutes?
Heartbeats fire on interval, not immediately.

### "Heartbeat runs but no message"

- `target: "none"` means run but don't deliver — change to `"last"` or specific channel
- `HEARTBEAT_OK`-only replies are dropped — this is correct behavior

### "Heartbeat context is too big"

- Enable `lightContext: true` — only loads `HEARTBEAT.md`
- Enable `isolatedSession: true` — fresh session each time
- Keep `HEARTBEAT.md` small (< 500 tokens)

## Manual Wake (Testing)

Trigger heartbeat immediately:

```bash
# Immediate wake
openclaw system event --text "Check for urgent follow-ups" --mode now

# Next scheduled tick
openclaw system event --text "Check for urgent follow-ups" --mode next-heartbeat
```

If multiple agents have `heartbeat` configured, manual wake runs all of them.

## Heartbeat vs Cron: When to Use Each

| Use Case | Heartbeat | Cron |
|----------|-----------|------|
| Multiple checks batched | ✅ Yes | ❌ No (one task per cron) |
| Needs conversational context | ✅ Yes | ❌ No (isolated) |
| Timing can drift (~30 min) | ✅ Yes | ❌ Needs exact timing |
| Exact schedule required | ❌ No | ✅ Yes |
| One-shot reminders | ❌ No | ✅ Yes |
| Different model/thinking level | ❌ No | ✅ Yes |
| Deliver without main session | ❌ No | ✅ Yes |

**Pro tip:** Batch similar periodic checks into `HEARTBEAT.md` instead of creating multiple cron jobs.

## Real-World Example: DocLoop

```json5
{
  agents: {
    list: [
      {
        id: "docloop",
        name: "DocLoop",
        workspace: "~/.openclaw/workspace-docloop",
        heartbeat: {
          every: "30m",
          target: "discord",
          accountId: "docloop",
          to: "1488561209300095230",
          lightContext: true,
          isolatedSession: true,
          prompt: "Read HEARTBEAT.md if it exists. Follow the DocLoop 6-Phase Workflow strictly. Do not infer tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        },
      },
    ],
  },
}
```

**HEARTBEAT.md:**
```markdown
# DocLoop Documentation Maintenance

## 6-Phase Checklist
- [ ] PHASE 1: Scan for new releases, skills, use cases
- [ ] PHASE 2: Audit gaps against existing docs
- [ ] PHASE 3: Research authoritative sources
- [ ] PHASE 4: Create/update documentation
- [ ] PHASE 5: Validate (keep-or-discard gate)
- [ ] PHASE 6: Commit & report

## Rules
1. Official docs = ground truth only — NO invention
2. Never push to main — always PR
3. Failed validation → PENDING_REVIEW.md
```

## Related Modules

- Previous: [11-creating-agents](../11-creating-agents/)
- Automation: [06-automation](../06-automation/)
- Cron Jobs: [06-automation](../06-automation/) (see Cron vs Heartbeat section)

---

**Time to complete:** 15 minutes
**Prerequisites:** Working Gateway, one channel configured
**Outcome:** Proactive agent check-ins for maintenance and alerts

**Status:** ✅ VALIDATED — Phase 5 Complete
**DocLoop Phase:** 6 Ready (Commit & Report)
