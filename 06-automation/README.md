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

## Quick Start

### Create Your First Cron Job

```bash
# Add a daily briefing
openclaw cron add "daily-briefing" \
  --schedule "0 9 * * *" \
  --command "morning briefing" \
  --channel telegram
```

### List Cron Jobs

```bash
openclaw cron list
```

Output:
```
NAME              SCHEDULE        NEXT RUN        STATUS
daily-briefing    0 9 * * *       Tomorrow 9:00   active
weekly-review     0 17 * * 5      Fri 5:00pm      active
```

### Remove a Job

```bash
openclaw cron remove daily-briefing
```

## Cron Syntax

| Expression | Meaning |
|------------|---------|
| `0 9 * * *` | Every day at 9:00 AM |
| `0 */6 * * *` | Every 6 hours |
| `0 9 * * 1` | Every Monday at 9:00 AM |
| `0 0 1 * *` | First day of every month |
| `*/15 * * * *` | Every 15 minutes |

Format: `minute hour day month weekday`

## Common Automation Patterns

### Morning Briefing

```bash
openclaw cron add "morning-brief" \
  --schedule "0 8 * * *" \
  --command "Send morning briefing: weather, calendar, and priority tasks" \
  --channel telegram
```

### Daily Standup Reminder

```bash
openclaw cron add "standup" \
  --schedule "0 9 * * 1-5" \
  --command "Daily standup time! Check GitHub PRs and report status." \
  --channel slack
```

### Weekly Review

```bash
openclaw cron add "weekly-review" \
  --schedule "0 17 * * 5" \
  --command "Weekly review: summarize completed tasks, check upcoming priorities" \
  --channel discord
```

### Health Check

```bash
openclaw cron add "health-check" \
  --schedule "0 */4 * * *" \
  --command "Run health check on all services and report any issues" \
  --channel slack
```

## Copy-paste examples

### Cron job configuration file

```yaml
# ~/.openclaw/cron.yaml
jobs:
  morning-briefing:
    schedule: "0 8 * * *"
    command: "morning briefing"
    channel: telegram
    
  daily-backup:
    schedule: "0 2 * * *"
    command: "backup data"
    enabled: false  # Disabled by default
    
  weekly-report:
    schedule: "0 9 * * 1"
    command: "generate weekly report"
    channel: email
    parameters:
      format: markdown
```

### Automation skill script

```javascript
// ~/.openclaw/skills/automation/scripts/briefing.js
module.exports = {
  async generateBriefing() {
    const weather = await skills.weather.getCurrent({ location: 'local' });
    const calendar = await skills.calendar.getToday();
    const tasks = await skills.todo.getPending();
    
    const message = `🌤️ Good morning!

**Weather:** ${weather.condition}, ${weather.temp}°C

**Today's Events:**
${calendar.events.map(e => `- ${e.time}: ${e.title}`).join('\n')}

**Pending Tasks:** ${tasks.length}
${tasks.slice(0, 3).map(t => `- ${t.title}`).join('\n')}
`;
    
    return message;
  }
};
```

### Conditional automation

```javascript
// Only run on work days
function isWorkDay() {
  const day = new Date().getDay();
  return day >= 1 && day <= 5; // Monday-Friday
}

module.exports = {
  async conditionalTask() {
    if (!isWorkDay()) {
      return 'Skipped (not a work day)';
    }
    // ... do the work
  }
};
```

## Common mistakes and troubleshooting

| Problem | Solution |
|---------|----------|
| Job not running | Check gateway is running; verify schedule syntax |
| Wrong timezone | Set `TZ` environment variable |
| Job runs but no output | Verify channel is connected; check permissions |
| Missed executions | Gateway was down; jobs don't backfill by default |

## Advanced patterns

### Dynamic scheduling

```javascript
// Schedule a one-time reminder
openclaw cron add "reminder-123" \
  --schedule "0 14 25 12 *" \
  --command "Meeting with team" \
  --once  # Remove after execution
```

### Job dependencies

```yaml
jobs:
  fetch-data:
    schedule: "0 */6 * * *"
    command: "fetch metrics"
    
  process-data:
    schedule: "5 */6 * * *"  # 5 min after fetch
    command: "process metrics"
```

### Error notifications

```javascript
// Auto-notify on failure
module.exports = {
  async monitoredTask() {
    try {
      await riskyOperation();
    } catch (error) {
      await skills.notify.send({
        channel: 'telegram',
        message: `⚠️ Task failed: ${error.message}`
      });
      throw error;
    }
  }
};
```

## Related modules and next step

- Previous: [05-integrations](../05-integrations/)
- Next: [07-browser-automation](../07-browser-automation/)
- Reference: [Cron documentation](https://docs.openclaw.ai/automation/cron-jobs)

---

**Time to complete:** 45 minutes  
**Prerequisites:** [05-integrations](../05-integrations/)  
**Outcome:** Automated recurring tasks, scheduled briefings, hands-off workflows
