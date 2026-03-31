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

### Simple Workflow

```javascript
// ~/.openclaw/skills/morning-routine/scripts/run.js
module.exports = {
  async execute() {
    // 1. Gather data
    const weather = await skills.weather.getCurrent();
    const events = await skills.calendar.getToday();
    
    // 2. Process
    const briefing = {
      greeting: weather.temp > 20 ? 'Good morning! ☀️' : 'Good morning! 🧥',
      weather: `${weather.condition}, ${weather.temp}°C`,
      events: events.length
    };
    
    // 3. Deliver
    await skills.notify.send({
      channel: 'telegram',
      message: `${briefing.greeting}\n\n${briefing.weather}\n\nYou have ${briefing.events} events today.`
    });
    
    // 4. Log
    await skills.memory.saveNote('daily-briefing', briefing);
    
    return 'Briefing sent and logged';
  }
};
```

### Complex Workflow with Branches

```javascript
// ~/.openclaw/skills/pr-manager/scripts/handle.js
module.exports = {
  async handlePRs() {
    const prs = await skills.github.getPendingReviews();
    
    for (const pr of prs) {
      // Decision branch
      if (pr.size === 'large') {
        // Complex PR: request detailed review
        await skills.notify.send({
          channel: 'slack',
          message: `Large PR #${pr.number} needs thorough review: ${pr.url}`
        });
      } else if (pr.testsPassing) {
        // Clean small PR: auto-approve option
        const approval = await skills.approval.request({
          message: `Approve small PR #${pr.number}?`,
          timeout: '1h'
        });
        
        if (approval) {
          await skills.github.approve(pr.number);
        }
      } else {
        // Failing tests: notify author
        await skills.github.comment(pr.number, '@' + pr.author + ' Tests are failing');
      }
    }
  }
};
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
    await skills.memory.saveNote('workflow-log', {
      timestamp: new Date().toISOString(),
      result
    });
  },
  
  async handleError(error) {
    await skills.notify.send({
      channel: 'telegram',
      message: `⚠️ Workflow failed: ${error.message}`
    });
  }
};
```

### Multi-channel workflow

```javascript
// ~/.openclaw/skills/alert-system/scripts/escalate.js
module.exports = {
  async escalate({ severity, message, details }) {
    const channels = {
      low: ['telegram'],
      medium: ['telegram', 'email'],
      high: ['telegram', 'email', 'slack', 'sms']
    };
    
    const targets = channels[severity] || channels.low;
    
    for (const channel of targets) {
      await skills.notify.send({
        channel,
        message: `[${severity.toUpperCase()}] ${message}`,
        details
      });
    }
  }
};
```

### Scheduled workflow with cron

```yaml
# ~/.openclaw/cron.yaml
jobs:
  morning-routine:
    schedule: "0 8 * * *"
    command: "run skill morning-routine"
    
  pr-check:
    schedule: "0 9,14 * * 1-5"
    command: "run skill pr-manager"
    
  weekly-report:
    schedule: "0 17 * * 5"
    command: "run skill weekly-summary"
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

```javascript
// Chain workflows together
module.exports = {
  async masterWorkflow() {
    const step1 = await skills.workflow1.execute();
    if (!step1.success) return step1;
    
    const step2 = await skills.workflow2.execute(step1.result);
    if (!step2.success) return step2;
    
    return skills.workflow3.execute(step2.result);
  }
};
```

### Dynamic workflow

```javascript
// Build workflow based on context
module.exports = {
  async dynamicWorkflow(context) {
    const steps = [];
    
    if (context.needsResearch) {
      steps.push(skills.research.gather);
    }
    
    if (context.needsApproval) {
      steps.push(skills.approval.request);
    }
    
    for (const step of steps) {
      await step(context);
    }
  }
};
```

## Related modules and next step

- Previous: [07-browser-automation](../07-browser-automation/)
- Next: [09-security-and-ops](../09-security-and-ops/)
- Examples: [../examples/](../examples/)

---

**Time to complete:** 90 minutes  
**Prerequisites:** [07-browser-automation](../07-browser-automation/)  
**Outcome:** Complete automated systems handling real-world scenarios
