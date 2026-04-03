# PR Alerts Workflow

> Get notified about pull requests that need your attention — directly in Slack or Discord.

## What it does

Monitors GitHub for:
- PRs awaiting your review
- PRs where you're mentioned
- PRs that have been updated
- PRs with failing checks

Notifies you via your preferred channel.

## Components Used

| Component | Purpose |
|-----------|---------|
| **GitHub skill** | Fetches PR data |
| **Discord/Slack** | Delivers notifications |
| **Cron** | Checks periodically |
| **Webhook** | Instant PR notifications |

## Setup

### 1. Install GitHub Skill

```bash
openclaw skill install github
```

### 2. Configure GitHub

```bash
# Create personal access token at https://github.com/settings/tokens
# Required scopes: repo, read:user

openclaw config set github.token YOUR_GITHUB_TOKEN
```

### 3. Create the Workflow

Create `~/.openclaw/skills/pr-alerts/scripts/check.js`:

```javascript
module.exports = {
  async checkPRs(options = {}) {
    const { 
      repos = ['owner/repo'], 
      filter = 'review_requested',
      notify = true 
    } = options;
    
    const alerts = [];
    
    for (const repo of repos) {
      const prs = await skills.github.getPullRequests({
        repo,
        state: 'open',
        filter
      });
      
      for (const pr of prs) {
        // Check if we haven't notified about this recently
        const cacheKey = `pr-${pr.number}`;
        const lastNotified = await this.getLastNotified(cacheKey);
        
        if (!lastNotified || pr.updated_at > lastNotified) {
          alerts.push({
            repo,
            number: pr.number,
            title: pr.title,
            author: pr.user.login,
            url: pr.html_url,
            updated: pr.updated_at,
            checks: pr.checks?.state || 'unknown'
          });
          
          await this.setLastNotified(cacheKey, new Date().toISOString());
        }
      }
    }
    
    if (notify && alerts.length > 0) {
      await this.sendNotifications(alerts);
    }
    
    return alerts;
  },
  
  async sendNotifications(alerts) {
    const message = this.formatMessage(alerts);
    
    await skills.notify.send({
      channel: 'slack',  // or 'discord'
      message
    });
  },
  
  formatMessage(alerts) {
    const header = `🔔 **${alerts.length} PR${alerts.length > 1 ? 's' : ''} need attention**\n\n`;
    
    const items = alerts.map(pr => {
      const status = pr.checks === 'success' ? '✅' : 
                     pr.checks === 'failure' ? '❌' : '⏳';
      return `${status} **${pr.repo}#${pr.number}**: ${pr.title}\n   By @${pr.author} | [View PR](${pr.url})`;
    }).join('\n\n');
    
    return header + items;
  },
  
  // Simple cache using memory
  async getLastNotified(key) {
    const data = await skills.memory.get(`pr-alerts:${key}`);
    return data?.timestamp;
  },
  
  async setLastNotified(key, timestamp) {
    await skills.memory.save(`pr-alerts:${key}`, { timestamp });
  }
};
```

### 4. Add SKILL.md

Create `~/.openclaw/skills/pr-alerts/SKILL.md`:

```markdown
# PR Alerts

## Description
Monitor GitHub PRs and notify about items needing attention.

## Tools

### checkPRs
Check for PRs requiring review.

**Parameters:**
- repos (array): Repository list ["owner/repo"]
- filter (string): Filter type (review_requested, mentions, etc.)
- notify (boolean): Send notifications (default: true)

**Example:**
```javascript
checkPRs({ 
  repos: ["myorg/frontend", "myorg/backend"],
  filter: "review_requested"
})
```

### sendNotifications
Send formatted notifications.

**Parameters:**
- alerts (array): PR data to notify about
```

### 5. Set Up Cron

```bash
# Check every hour during work hours
openclaw cron add "pr-alerts" \
  --schedule "0 9-18 * * 1-5" \
  --command "run skill pr-alerts checkPRs" \
  --channel slack
```

### 6. Optional: GitHub Webhook

For instant notifications (instead of polling):

1. Create webhook receiver:

```javascript
// ~/.openclaw/skills/pr-alerts/scripts/webhook.js
module.exports = {
  async handleWebhook(payload) {
    if (payload.action === 'review_requested') {
      // Check if we're the requested reviewer
      if (payload.requested_reviewer?.login === process.env.GITHUB_USERNAME) {
        await this.sendNotification({
          title: `🆕 Review requested on #${payload.pull_request.number}`,
          pr: payload.pull_request
        });
      }
    }
  }
};
```

2. Configure webhook URL in GitHub repo settings.

## Usage

### Automatic (Cron)

Checks run automatically during work hours.

### Manual Check

```
User: Check my PRs
Agent: [Fetches and displays open PRs]
```

### Custom Check

```
User: Check PRs in the backend repo
Agent: [Checks specific repo]
```

## Customization

### Multiple Teams

```javascript
// Check different repos for different teams
const teamConfigs = {
  frontend: ['myorg/web', 'myorg/mobile'],
  backend: ['myorg/api', 'myorg/workers'],
  platform: ['myorg/infrastructure']
};

async checkTeamPRs(team) {
  const repos = teamConfigs[team];
  return this.checkPRs({ repos });
}
```

### Smart Prioritization

```javascript
// Prioritize based on age and checks
function prioritize(alerts) {
  return alerts.sort((a, b) => {
    // Failing checks first
    if (a.checks === 'failure' && b.checks !== 'failure') return -1;
    if (b.checks === 'failure' && a.checks !== 'failure') return 1;
    
    // Then by age (oldest first)
    return new Date(a.updated) - new Date(b.updated);
  });
}
```

### Weekend Digest

```bash
# Friday evening summary
openclaw cron add "pr-weekend" \
  --schedule "0 17 * * 5" \
  --command "run skill pr-alerts checkPRs --notify=false && generate report" \
  --channel email
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| No PRs found | Check token scopes (need 'repo') |
| Wrong repos | Verify 'owner/repo' format |
| Duplicate notifications | Check cache is working |
| Notifications not sending | Verify channel connection |

## Related

- [05-integrations](../../05-integrations/) - GitHub setup
- [06-automation](../../06-automation/) - Cron configuration
- [08-workflows](../../08-workflows/) - More workflow patterns

---

**Difficulty:** 🟡 Medium  
**Setup time:** 30 minutes  
**Maintenance:** Low
