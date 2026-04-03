# 06 - Automation

> Set up recurring tasks, briefings, and reminders with OpenClaw's built-in cron.

## What this module solves

Cron is OpenClaw's built-in scheduler. It persists jobs, wakes the agent at the right time, and executes tasks — even if you're not actively chatting.

## When to use this module

- You want daily/weekly briefings
- You need recurring reminders
- You want automated health checks
- You want scheduled data collection

## How Cron Works

```
┌─────────────┐     ┌──────────┐     ┌─────────────┐
│  Cron Job   │────►│  Gateway │────►│  Agent Run  │
│  (scheduled)│     │  (wakes) │     │  (executes) │
└─────────────┘     └──────────┘     └─────────────┘
                                           │
                                           ▼
                                    ┌─────────────┐
                                    │   Channel   │
                                    │  (notifies) │
                                    └─────────────┘
```

## Cron Commands Overview

```bash
# List all cron jobs
openclaw cron list

# Add a recurring job
openclaw cron add \
  --name "job-name" \
  --cron "0 9 * * *" \
  --message "What to do"

# Add a one-shot job
openclaw cron add \
  --name "reminder" \
  --at "2026-04-05 14:00" \
  --message "Meeting in 15 minutes"

# Edit a job
openclaw cron edit <job-id> --announce --channel telegram

# Enable/disable a job
openclaw cron enable <job-id>
openclaw cron disable <job-id>

# Run a job manually
openclaw cron run <job-id>

# Check job runs
openclaw cron runs --id <job-id>

# Remove a job
openclaw cron rm <job-id>
```

## Quick Start

### Create Your First Cron Job

```bash
# Add a daily briefing with delivery to Telegram
openclaw cron add \
  --name "daily-briefing" \
  --cron "0 9 * * *" \
  --message "Send morning briefing with weather, calendar, and priority tasks" \
  --announce \
  --channel telegram \
  --to "123456789"
```

### List Cron Jobs

```bash
openclaw cron list
```

Output shows job ID, name, schedule, next run, and status.

### Check Job Runs

```bash
# Check outcomes of a specific job
openclaw cron runs --id <job-id>
```

### Remove a Job

```bash
openclaw cron rm <job-id>
```

## Cron Syntax

The `--cron` flag uses standard cron expression format:

| Expression | Meaning |
|------------|---------|
| `0 9 * * *` | Every day at 9:00 AM |
| `0 */6 * * *` | Every 6 hours |
| `0 9 * * 1` | Every Monday at 9:00 AM |
| `0 0 1 * *` | First day of every month |
| `*/15 * * * *` | Every 15 minutes |

Format: `minute hour day month weekday`

## Session Types

Cron jobs can run in different session modes:

| Session Type | Description | Use Case |
|--------------|-------------|----------|
| `main` | Runs during heartbeat, shares context | Silent tasks, background work |
| `isolated` | Fresh session per run (default) | Lightweight jobs, no history needed |
| `session:custom-id` | Persistent named session | Jobs that need to remember state |

### Session Examples

```bash
# Main session - runs during heartbeat
openclaw cron add \
  --name "health-check" \
  --cron "0 */4 * * *" \
  --message "Check service health" \
  --session main

# Isolated session - fresh context each run (default)
openclaw cron add \
  --name "morning-brief" \
  --cron "0 8 * * *" \
  --message "Generate morning briefing" \
  --session isolated \
  --announce \
  --channel telegram

# Custom persistent session
openclaw cron add \
  --name "data-collector" \
  --cron "0 */6 * * *" \
  --message "Collect metrics" \
  --session session:metrics-collector
```

## Common Automation Patterns

### Morning Briefing

```bash
openclaw cron add \
  --name "morning-brief" \
  --cron "0 8 * * *" \
  --message "Send morning briefing: weather, calendar, and priority tasks" \
  --announce \
  --channel telegram \
  --to "YOUR_CHAT_ID"
```

### Daily Standup Reminder

```bash
openclaw cron add \
  --name "standup" \
  --cron "0 9 * * 1-5" \
  --message "Daily standup time! Check GitHub PRs and report status." \
  --announce \
  --channel slack \
  --to "#standup-channel"
```

### Weekly Review

```bash
openclaw cron add \
  --name "weekly-review" \
  --cron "0 17 * * 5" \
  --message "Weekly review: summarize completed tasks, check upcoming priorities" \
  --announce \
  --channel discord \
  --to "YOUR_USER_ID"
```

### Health Check

```bash
openclaw cron add \
  --name "health-check" \
  --cron "0 */4 * * *" \
  --message "Run health check on all services and report any issues" \
  --session isolated \
  --no-deliver
```

### One-Shot Reminder

```bash
# Schedule a one-time reminder
openclaw cron add \
  --name "meeting-reminder" \
  --at "2026-04-05 14:00" \
  --message "Meeting with team in 15 minutes" \
  --announce \
  --channel telegram
```

## Copy-Paste Examples

### Job with Light Context

For faster execution with minimal bootstrap:

```bash
openclaw cron add \
  --name "quick-check" \
  --cron "0 * * * *" \
  --message "Quick status check" \
  --light-context \
  --no-deliver
```

### Editing Jobs

```bash
# Add delivery to an existing job
openclaw cron edit <job-id> \
  --announce \
  --channel telegram \
  --to "123456789"

# Disable delivery
openclaw cron edit <job-id> --no-deliver

# Enable light context
openclaw cron edit <job-id> --light-context
```

### Manual Run

```bash
# Trigger a job manually
openclaw cron run <job-id>
# Returns: { ok: true, enqueued: true, runId }

# Check the outcome
openclaw cron runs --id <job-id>
```

## Retry Policy

Cron jobs have automatic retry on failure:

| Attempt | Delay |
|---------|-------|
| 1st retry | 30 seconds |
| 2nd retry | 1 minute |
| 3rd retry | 5 minutes |
| 4th retry | 15 minutes |
| 5th retry | 60 minutes |

Jobs that fail after all retries are marked as failed. Check `openclaw cron runs --id <job-id>` for details.

## Job Storage

Cron jobs are stored at:
- **Location:** `~/.openclaw/cron/jobs.json`
- **Format:** JSON with job definitions
- **Persistence:** Survives restarts

## Common Mistakes and Troubleshooting

| Problem | Solution |
|---------|----------|
| Job not running | Check gateway is running: `openclaw gateway status` |
| Wrong timezone | Set `TZ` environment variable |
| Job runs but no output | Verify `--announce` flag and channel connection |
| Missed executions | Gateway was down; jobs don't backfill by default |
| "Invalid cron expression" | Check format: `minute hour day month weekday` |
| Can't edit job | Use job ID from `openclaw cron list`, not name |

## Advanced Patterns

## Task Flow (v2026.4.2+) — Multi-Step Workflow Orchestration

> **New in v2026.4.2:** Task Flow is the orchestration layer above background tasks for durable multi-step workflows.

### When to Use Task Flow vs Cron

| Use Case | Tool |
|----------|------|
| Single recurring job | Cron |
| Multi-step pipeline (A → B → C) | Task Flow (managed) |
| Track externally created tasks | Task Flow (mirrored) |
| Simple daily briefing | Cron |
| Complex report generation | Task Flow |

### How Task Flow Works

Task Flow coordinates multiple tasks with:
- **Durable state:** Progress survives gateway restarts
- **Revision tracking:** Conflict detection for concurrent updates
- **Sync modes:** Managed (controls lifecycle) or Mirrored (observes external tasks)

```
┌─────────────┐     ┌──────────┐     ┌─────────────┐
│   Flow      │────►│  Step 1  │────►│  Step 2     │
│ (orchestrates)│    │  Task    │     │  Task       │
└─────────────┘     └──────────┘     └─────────────┘
       │                                    │
       └────────────────────────────────────┘
                    (waits, then continues)
```

### CLI Commands

```bash
# List active and recent flows
openclaw tasks flow list

# Inspect a specific flow
openclaw tasks flow <lookup>

# Cancel a running flow
openclaw tasks flow cancel <lookup>

# Inspect individual tasks
openclaw tasks list
```

### Sync Modes Explained

**Managed Mode:** Task Flow creates and drives tasks automatically.
```
Flow: weekly-report
  Step 1: gather-data     → task created → succeeded
  Step 2: generate-report → task created → running
  Step 3: deliver         → pending
```

**Mirrored Mode:** Task Flow observes tasks created by cron/CLI without controlling them.
```
Flow: morning-ops
  Cron Job A: backups      → observed → succeeded
  Cron Job B: health-check → observed → running
  Cron Job C: report       → observed → pending
```

### Example: Weekly Report Flow

A managed flow that coordinates 3 sequential tasks:

1. **Gather data** (task 1)
2. **Generate report** (task 2) — starts after task 1 succeeds
3. **Deliver to Slack** (task 3) — starts after task 2 succeeds

If the gateway restarts during step 2, Task Flow resumes from that point.

### Durable State & Recovery

Task Flow persists its state to SQLite:
- **Location:** `$OPENCLAW_STATE_DIR/tasks/flows.sqlite`
- **Retention:** Terminal flows kept for 7 days
- **Recovery:** Automatic resume after gateway restart

### Task Flow vs Background Tasks

| Feature | Background Task | Task Flow |
|---------|-----------------|-----------|
| Unit of work | Single operation | Multi-step coordination |
| State | Task-level | Flow-level with step tracking |
| Recovery | Manual restart | Automatic resume |
| Use case | One-shot job | Complex workflows |

### Common Patterns

**Weekly Report Pipeline:**
```
Flow: weekly-report (managed)
├── Step 1: collect-metrics (cron job)
├── Step 2: analyze-data (task)
├── Step 3: generate-pdf (task)
└── Step 4: send-email (task)
```

**Morning Operations Dashboard:**
```
Flow: morning-ops (mirrored)
├── Cron: backup-check
├── Cron: dependency-scan
├── Cron: health-check
└── Cron: daily-briefing
```

### Troubleshooting Task Flow

| Problem | Solution |
|---------|----------|
| Flow stuck | Check `openclaw tasks flow list` for failed steps |
| Can't cancel | Use lookup ID from list, not flow name |
| Lost after restart | Wait 30s for reconciliation, check `openclaw tasks flow list` |
| Step keeps failing | Check individual task with `openclaw tasks list` |

---

## Copy-Paste Examples

### Job with Light Context

For faster execution with minimal bootstrap:

```bash
openclaw cron add \
  --name "quick-check" \
  --cron "0 * * * *" \
  --message "Quick status check" \
  --light-context \
  --no-deliver
```

### Editing Jobs

```bash
# Add delivery to an existing job
openclaw cron edit <job-id> \
  --announce \
  --channel telegram \
  --to "123456789"

# Disable delivery
openclaw cron edit <job-id> --no-deliver

# Enable light context
openclaw cron edit <job-id> --light-context
```

### Manual Run

```bash
# Trigger a job manually
openclaw cron run <job-id>
# Returns: { ok: true, enqueued: true, runId }

# Check the outcome
openclaw cron runs --id <job-id>
```

## Retry Policy

Cron jobs have automatic retry on failure:

| Attempt | Delay |
|---------|-------|
| 1st retry | 30 seconds |
| 2nd retry | 1 minute |
| 3rd retry | 5 minutes |
| 4th retry | 15 minutes |
| 5th retry | 60 minutes |

Jobs that fail after all retries are marked as failed. Check `openclaw cron runs --id <job-id>` for details.

## Job Storage

Cron jobs are stored at:
- **Location:** `~/.openclaw/cron/jobs.json`
- **Format:** JSON with job definitions
- **Persistence:** Survives restarts

## Common Mistakes and Troubleshooting

| Problem | Solution |
|---------|----------|
| Job not running | Check gateway is running: `openclaw gateway status` |
| Wrong timezone | Set `TZ` environment variable |
| Job runs but no output | Verify `--announce` flag and channel connection |
| Missed executions | Gateway was down; jobs don't backfill by default |
| "Invalid cron expression" | Check format: `minute hour day month weekday` |
| Can't edit job | Use job ID from `openclaw cron list`, not name |

## Advanced Patterns

### Job Dependencies

Chain jobs with offset schedules:

```bash
# Fetch data every 6 hours
openclaw cron add \
  --name "fetch-data" \
  --cron "0 */6 * * *" \
  --message "Fetch metrics from API" \
  --no-deliver

# Process data 5 minutes after fetch
openclaw cron add \
  --name "process-data" \
  --cron "5 */6 * * *" \
  --message "Process collected metrics" \
  --announce \
  --channel slack
```

### Conditional Logic in Messages

The cron `--message` is a prompt to the agent. You can include conditional instructions:

```bash
openclaw cron add \
  --name "smart-briefing" \
  --cron "0 9 * * *" \
  --message "Check calendar and weather. If it's raining, mention umbrella. If calendar is empty, suggest a focus day." \
  --announce \
  --channel telegram
```

### Cron Status Check

```bash
# Check overall cron system status
openclaw cron status
```

## Related Modules and Next Steps

- Previous: [05-integrations](../05-integrations/)
- Next: [07-browser-automation](../07-browser-automation/)
- Reference: [Cron documentation](https://docs.openclaw.ai/cli/cron)

---

**Time to complete:** 45 minutes  
**Prerequisites:** [05-integrations](../05-integrations/)  
**Outcome:** Automated recurring tasks, scheduled briefings, hands-off workflows
