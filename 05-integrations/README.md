# 05 - Integrations

> Connect OpenClaw to your tools: GitHub, Calendar, Email, Databases, and APIs.

## What this module solves

Skills are building blocks; integrations are how you connect to external services. This module covers the most useful integrations and how to configure them.

## When to use this module

- You want to automate GitHub workflows
- You need calendar management
- You want to send/receive emails
- You need database access

## Quick Start

### GitHub Integration

```bash
# Install GitHub skill
openclaw skill install github

# Configure with personal access token
openclaw config set github.token YOUR_GITHUB_TOKEN
```

Use it:

```
User: Check my PRs
Agent: You have 3 open PRs: ...

User: Review #42 in openclaw/openclaw
Agent: [Fetches PR details and provides summary]
```

### Calendar Integration

```bash
# Install calendar skill
openclaw skill install calendar

# Configure (Google Calendar example)
openclaw config set calendar.provider google
openclaw config set calendar.credentials ~/.openclaw/secrets/gcal.json
```

Use it:

```
User: What's on my calendar today?
Agent: You have 2 events: 10:00 Team Standup, 14:00 Design Review

User: Schedule a meeting with Alice tomorrow at 2pm
Agent: Created event "Meeting with Alice" for tomorrow at 14:00.
```

### Email Integration

```bash
# Install email skill
openclaw skill install email

# Configure SMTP/IMAP or use AgentMail
openclaw config set email.provider agentmail
openclaw config set email.api_key YOUR_AGENTMAIL_KEY
```

Use it:

```
User: Check my email
Agent: You have 5 unread messages. 2 require action.

User: Send email to team@company.com about the release
Agent: Draft ready. Send it? [Yes/No]
```

## Popular Integrations

| Integration | Use Case | Install Command |
|-------------|----------|-----------------|
| **github** | PRs, issues, repos | `openclaw skill install github` |
| **calendar** | Schedule, events | `openclaw skill install calendar` |
| **email** | Send/receive | `openclaw skill install email` |
| **notion** | Notes, wikis | `openclaw skill install notion` |
| **trello** | Project mgmt | `openclaw skill install trello` |
| **linear** | Issue tracking | `openclaw skill install linear` |
| **postgres** | Database access | `openclaw skill install postgres` |
| **sqlite** | Local database | `openclaw skill install sqlite` |

## Configuration Patterns

### Using Environment Variables

```bash
# ~/.bashrc or ~/.zshrc
export GITHUB_TOKEN=ghp_xxxxxxxx
export OPENAI_API_KEY=sk-xxxxxxxx

# Then reference in config
openclaw config set github.token "$GITHUB_TOKEN"
```

### Using .env Files

```bash
# ~/.openclaw/.env
GITHUB_TOKEN=ghp_xxxxxxxx
CALENDAR_CREDENTIALS=~/.openclaw/secrets/calendar.json
```

### Secrets Management

```bash
# Create secrets directory
mkdir -p ~/.openclaw/secrets
chmod 700 ~/.openclaw/secrets

# Store sensitive files
cp service-account.json ~/.openclaw/secrets/
```

## Copy-paste examples

### GitHub webhook handler

```javascript
// ~/.openclaw/skills/github-webhook/scripts/handler.js
module.exports = {
  onPR: async (payload) => {
    const { action, pull_request } = payload;
    
    if (action === 'opened') {
      await skills.notify.send({
        channel: 'slack',
        message: `New PR: ${pull_request.title} by ${pull_request.user.login}`
      });
    }
  }
};
```

### Database query helper

```javascript
// ~/.openclaw/skills/db-query/scripts/query.js
const { Client } = require('pg');

module.exports = {
  async runQuery({ sql, params = [] }) {
    const client = new Client({
      connectionString: process.env.DATABASE_URL
    });
    
    await client.connect();
    const result = await client.query(sql, params);
    await client.end();
    
    return result.rows;
  }
};
```

## Common mistakes and troubleshooting

| Problem | Solution |
|---------|----------|
| 401 Unauthorized | Check API tokens; verify scopes/permissions |
| Rate limiting | Add delays; use caching; upgrade plan |
| Connection timeout | Check firewall; verify endpoint URLs |
| SSL errors | Update certificates; check system time |

## Advanced patterns

### Integration chaining

```javascript
// Skill that combines multiple integrations
module.exports = {
  async morningBriefing() {
    const weather = await skills.weather.getCurrent();
    const events = await skills.calendar.getToday();
    const prs = await skills.github.getReviewRequests();
    
    return {
      weather,
      calendar: events,
      github: prs
    };
  }
};
```

### Error handling wrapper

```javascript
// Robust integration call
async function safeIntegrationCall(fn, fallback) {
  try {
    return await fn();
  } catch (error) {
    console.error('Integration error:', error.message);
    return fallback;
  }
}
```

## Related modules and next step

- Previous: [04-skills](../04-skills/)
- Next: [06-automation](../06-automation/)
- Reference: [Integration catalog](CATALOG.md)

---

**Time to complete:** 45 minutes  
**Prerequisites:** [04-skills](../04-skills/)  
**Outcome:** Connected to external services, automated tool workflows
