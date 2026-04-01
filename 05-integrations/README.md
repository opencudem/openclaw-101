# 05 - Integrations

> Connect OpenClaw to your tools: GitHub, Calendar, Email, and more.

## What this module solves

Skills are building blocks; integrations are how you connect to external services. This module covers how to discover, install, and configure real integrations from ClawHub.

## When to use this module

- You want to automate GitHub workflows
- You need calendar management
- You want to send/receive emails
- You need database access

## Discovering Real Integrations

Always verify integrations exist on ClawHub before documenting:

```bash
# Search ClawHub for available skills
openclaw skills search "github"
openclaw skills search "calendar"
openclaw skills search "email"
openclaw skills search "database"

# List available skills with details
openclaw skills list --eligible
```

**ClawHub Registry:** https://clawhub.ai

## Verified Integrations

These integrations have been verified on ClawHub:

### GitHub Integration

```bash
# Install GitHub skill from ClawHub
openclaw skills install github

# Verify installation
openclaw skills info github
```

**What it does:** Uses the `gh` CLI to interact with GitHub (issues, PRs, CI runs)

**Requirements:**
- `gh` CLI must be installed and authenticated
- Run `gh auth login` to authenticate

Use it:

```
User: Check my PRs
Agent: You have 3 open PRs: ...

User: Review #42 in openclaw/openclaw
Agent: [Fetches PR details using gh pr view 42 --repo openclaw/openclaw]
```

---

### Calendar Integration

```bash
# Search for calendar skills
openclaw skills search "calendar"

# Install a verified calendar skill
openclaw skills install caldav-calendar

# Or install calendar-planner for advanced planning
openclaw skills install calendar-planner
```

**What it does:** Syncs and queries CalDAV calendars (iCloud, Google, Fastmail, Nextcloud)

**Requirements:**
- `vdirsyncer` and `khal` binaries
- CalDAV calendar credentials

Use it:

```
User: What's on my calendar today?
Agent: [Uses khal list to show today's events]

User: Schedule a meeting tomorrow at 2pm
Agent: [Uses khal new to create the event]
```

---

### Email Integration

```bash
# Search for email skills
openclaw skills search "email"

# Install IMAP/SMTP email skill
openclaw skills install imap-smtp-email
```

**What it does:** Read and send email via IMAP/SMTP protocols

**Requirements:**
- Node.js and npm
- IMAP/SMTP credentials
- Run `bash setup.sh` to configure

Use it:

```
User: Check my email
Agent: You have 5 unread messages. 2 require action.

User: Send email to team@company.com about the release
Agent: [Drafts and sends email via SMTP]
```

---

## Finding and Installing Skills

### Search ClawHub

```bash
# Search for skills by keyword
openclaw skills search "github"
openclaw skills search "weather"
openclaw skills search "database"

# Get detailed info about a skill
openclaw skills info github
```

### Install Skills

```bash
# Install from ClawHub
openclaw skills install <slug>

# Install specific version
openclaw skills install <slug> --version <version>

# Update a skill
openclaw skills update <slug>

# Check for issues with installed skills
openclaw skills check
```

**Install Location:** `~/.openclaw/skills/` (for managed/local skills)

---

## Configuration Patterns

### Using SecretRef for Secure Credentials

Instead of storing credentials in plain text config files, use SecretRef:

```yaml
# ~/.openclaw/config.yaml
skills:
  entries:
    github:
      config:
        token:
          secretRef: GITHUB_TOKEN  # Reads from OPENCLAW_SECRET_GITHUB_TOKEN env var
    email:
      config:
        smtp_password:
          secretRef: SMTP_PASSWORD
```

**SecretRef format:**
```json
{
  "source": "env",
  "provider": "default",
  "id": "API_KEY_NAME"
}
```

The agent will look for `OPENCLAW_SECRET_<id>` in environment variables.

### Using Environment Variables

```bash
# ~/.bashrc or ~/.zshrc
export GITHUB_TOKEN=ghp_xxxxxxxx
export OPENAI_API_KEY=sk-xxxxxxxx
export SMTP_PASSWORD=your_smtp_password
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

---

## Copy-Paste Examples

### Skill Discovery Workflow

```bash
# 1. Search for what you need
openclaw skills search "weather"

# 2. Get info about a specific skill
openclaw skills info weather

# 3. Install it
openclaw skills install weather

# 4. Verify it works
openclaw skills check
```

### GitHub Workflow Example

```bash
# Install and configure GitHub skill
openclaw skills install github

# Ensure gh CLI is authenticated
gh auth login

# Now you can ask your agent:
# "List my open PRs"
# "Check status of PR #42"
# "View failed CI runs"
```

### Database Query Helper

For database access, you may need to create a custom skill:

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

---

## Common Mistakes and Troubleshooting

| Problem | Solution |
|---------|----------|
| "skill not found" | Use `openclaw skills` (plural), not `openclaw skill` |
| 401 Unauthorized | Check API tokens; verify scopes/permissions |
| Rate limiting | Add delays; use caching; upgrade plan |
| Connection timeout | Check firewall; verify endpoint URLs |
| Skill not appearing | Run `openclaw skills check` to verify installation |
| Credentials exposed | Use SecretRef instead of plain text in config |

---

## Advanced Patterns

### Integration Chaining

Combine multiple skills in one workflow:

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

### Error Handling Wrapper

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

---

## Related Modules and Next Steps

- Previous: [04-skills](../04-skills/)
- Next: [06-automation](../06-automation/)
- Reference: [ClawHub](https://clawhub.ai)
- Command Reference: `openclaw skills --help`

---

**Time to complete:** 45 minutes  
**Prerequisites:** [04-skills](../04-skills/)  
**Outcome:** Connected to external services via verified ClawHub skills
