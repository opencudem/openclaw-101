# 08 - Workflows

> Combine channels, skills, integrations, and automation into complete systems.

## What this module solves

Individual features are useful; combined workflows are powerful. This module shows you how to build complete systems that handle real-world scenarios.

## When to use this module

- You want end-to-end automation
- You need multi-step processes
- You want conditional logic
- You need to coordinate multiple tools

## Workflow Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                       WORKFLOW                              │
├─────────────────────────────────────────────────────────────┤
│  Trigger → Process → Decision → Action → Notify → Archive  │
│                                                             │
│  Example:                                                   │
│  9:00 AM → Check PRs → Any pending? → Review → Slack → Log  │
└─────────────────────────────────────────────────────────────┘
```

## Quick Start

### Simple Workflow via Chat

Instead of writing JavaScript, use natural language:

```
User: Send me a morning briefing with weather and today's calendar
Agent: [Uses weather skill and calendar skill to generate briefing]

User: Schedule this briefing every day at 8 AM
Agent: [Creates cron job: openclaw cron add --name "morning-brief" --cron "0 8 * * *" --message "Send morning briefing" --announce --channel telegram]
```

### Complex Workflow with Branches via Chat

Use natural language for conditional logic:

```
User: Check my GitHub PRs
Agent: [Lists open PRs with details]

User: For large PRs, send detailed review requests to Slack
Agent: [Identifies large PRs, notifies Slack channel]

User: For small PRs with passing tests, ask me if I want to approve them
Agent: [Identifies small clean PRs, sends approval requests to your DMs]

User: For PRs with failing tests, notify the authors
Agent: [Comments on PRs mentioning authors about test failures]
```

Or set up automated cron jobs for different PR types:

```bash
# Large PRs - detailed review (every 2 hours)
openclaw cron add \
  --name "large-pr-review" \
  --cron "0 */2 * * 1-5" \
  --message "Find large PRs and request thorough reviews" \
  --announce \
  --channel slack

# Small clean PRs - approval queue (every hour)
openclaw cron add \
  --name "small-pr-approval" \
  --cron "0 * * * 1-5" \
  --message "Find small PRs with passing tests and queue for approval" \
  --announce \
  --channel telegram
```

## Copy-paste examples

### Workflow template

```javascript
// ~/.openclaw/skills/workflow-template/scripts/run.js
/**
 * Workflow: [Name]
 * Description: [What it does]
 * Trigger: [How it's triggered]
 */

module.exports = {
  async execute(context = {}) {
    try {
      // 1. INPUT - Gather
      const input = await this.gatherInput(context);
      
      // 2. PROCESS - Transform
      const result = await this.process(input);
      
      // 3. OUTPUT - Deliver
      await this.deliver(result);
      
      // 4. LOG - Record
      await this.log(result);
      
      return { success: true, result };
    } catch (error) {
      await this.handleError(error);
      return { success: false, error: error.message };
    }
  },
  
  async gatherInput(context) {
    return context;
  },
  
  async process(input) {
    return input;
  },
  
  async deliver(result) {
    console.log('Deliver:', result);
  },
  
  async log(result) {
    // Log to file instead of invented skills.memory
    const fs = require('fs').promises;
    await fs.appendFile(
      '/tmp/workflow.log',
      `${new Date().toISOString()}: ${JSON.stringify(result)}\n`
    );
  },
  
  async handleError(error) {
    // Use exec to send notification via openclaw CLI
    const { execSync } = require('child_process');
    execSync(`openclaw channels send --channel telegram --message "Workflow failed: ${error.message}"`);
  }
};
```

### Multi-channel workflow via cron and chat

Set up escalation via cron jobs:

```bash
# Low severity - Telegram only
openclaw cron add \
  --name "alert-low" \
  --cron "0 * * * *" \
  --message "Check low priority alerts and notify via telegram" \
  --announce \
  --channel telegram

# High severity - Multiple channels  
openclaw cron add \
  --name "alert-high" \
  --cron "*/15 * * * *" \
  --message "Check high priority alerts and notify via telegram, email, and slack" \
  --announce \
  --channel telegram
```

### Scheduled workflow with cron

Use `openclaw cron add` with descriptive messages:

```bash
# Morning briefing at 8 AM daily
openclaw cron add \
  --name "morning-routine" \
  --cron "0 8 * * *" \
  --message "Generate and send morning briefing with weather and calendar" \
  --announce \
  --channel telegram

# PR check at 9 AM and 2 PM on weekdays
openclaw cron add \
  --name "pr-check" \
  --cron "0 9,14 * * 1-5" \
  --message "Check pending GitHub PRs and send summary" \
  --announce \
  --channel slack

# Weekly report on Fridays at 5 PM
openclaw cron add \
  --name "weekly-report" \
  --cron "0 17 * * 5" \
  --message "Generate weekly activity summary" \
  --announce \
  --channel email
```

## Flagship Workflows

### Morning Briefing

- **Trigger:** Daily at 8:00 AM
- **Components:** Weather, Calendar, Tasks
- **Output:** Telegram message
- **Files:** [examples/personal-productivity/morning-briefing.md](../examples/personal-productivity/morning-briefing.md)

### Engineering Inbox

- **Trigger:** Cron + Webhook
- **Components:** GitHub, Discord, Linear
- **Output:** Channel notifications
- **Files:** [examples/engineering/pr-alerts.md](../examples/engineering/pr-alerts.md)

### Research Agent

- **Trigger:** Manual or scheduled
- **Components:** Browser, Search, Summarization
- **Output:** Report to email/Notion
- **Files:** [examples/research-and-content/research-agent.md](../examples/research-and-content/research-agent.md)

## Common mistakes and troubleshooting

| Problem | Solution |
|---------|----------|
| Workflow too complex | Break into smaller sub-workflows |
| Partial failures | Use transactions; implement rollback |
| Missing context | Add logging at each step |
| Race conditions | Add locks; use sequential execution |

## Advanced patterns

### Workflow composition

Chain multiple cron jobs or chat commands:

```
User: Run the morning briefing, then check my emails, then summarize my calendar
Agent: [Executes each workflow in sequence, passing context between steps]
```

Or create dependent cron jobs with delays:

```bash
# Step 1: Morning briefing at 8 AM
openclaw cron add \
  --name "step1-briefing" \
  --cron "0 8 * * *" \
  --message "Generate morning briefing" \
  --announce \
  --channel telegram

# Step 2: Email check at 8:05 AM (5 min after briefing)
openclaw cron add \
  --name "step2-email" \
  --cron "5 8 * * *" \
  --message "Check and summarize emails" \
  --announce \
  --channel telegram

# Step 3: Calendar summary at 8:10 AM
openclaw cron add \
  --name "step3-calendar" \
  --cron "10 8 * * *" \
  --message "Summarize today's calendar" \
  --announce \
  --channel telegram
```

### Dynamic workflow via conditional cron

Use different cron jobs based on conditions:

```bash
# Research workflow (runs if research-needed flag is set)
openclaw cron add \
  --name "research-check" \
  --cron "0 10 * * 1" \
  --message "If research-needed flag is set, perform web research and summarize findings" \
  --announce \
  --channel telegram

# Approval workflow (send to restricted channel for human approval)
openclaw cron add \
  --name "approval-check" \
  --cron "0 */4 * * *" \
  --message "Send pending approval requests to admin channel" \
  --announce \
  --channel telegram \
  --to "admin-user-id"
```

## Related modules and next step

- Previous: [07-browser-automation](../07-browser-automation/)
- Next: [09-security-and-ops](../09-security-and-ops/)
- Examples: [../examples/](../examples/)

---

**Time to complete:** 90 minutes  
**Prerequisites:** [07-browser-automation](../07-browser-automation/)  
**Outcome:** Complete automated systems handling real-world scenarios
