# 05 - Integrations

> Connect OpenClaw to your external tools and services: GitHub, Calendar, Email, and more.

## What this module solves

Integrations allow OpenClaw to interact with your external services - GitHub repos, Google Calendar, email accounts, databases, and APIs. This module covers how to configure these connections securely.

## When to use this module

- You want to automate GitHub workflows (PRs, issues, CI)
- You need calendar access (Google, iCloud, Outlook)
- You want to send/receive emails
- You need database connections
- You want API integrations (Slack, Notion, etc.)

## How Integrations Work

Integrations are configured via:
1. **Environment variables** - For API keys and tokens
2. **OpenClaw config** - `~/.openclaw/config.yaml` or `openclaw.json`
3. **Skill configs** - Individual skill configuration files

The agent uses these credentials to authenticate with external services.

---

## Configuration Fundamentals

### 1. Environment Variables (Recommended for tokens)

Store sensitive credentials in your shell:

```bash
# ~/.bashrc or ~/.zshrc
export GITHUB_TOKEN="ghp_your_token_here"
export OPENAI_API_KEY="sk-your_key_here"
export ANTHROPIC_API_KEY="sk-ant-your_key_here"
export SMTP_PASSWORD="your_email_password"
export CALENDAR_PASSWORD="your_calendar_app_password"
```

Then reload:
```bash
source ~/.bashrc
```

### 2. SecretRef in Config (Most Secure)

Use `secretRef` to reference environment variables in config:

```yaml
# ~/.openclaw/config.yaml
models:
  providers:
    anthropic:
      apiKey:
        secretRef: ANTHROPIC_API_KEY  # Reads from env var
        
skills:
  entries:
    github:
      config:
        token:
          secretRef: GITHUB_TOKEN
```

**SecretRef format:**
```yaml
key:
  secretRef: ENV_VAR_NAME  # Agent looks for OPENCLAW_SECRET_ENV_VAR_NAME or ENV_VAR_NAME
```

### 3. Direct Config (Not recommended for production)

For non-sensitive config only:

```yaml
# ~/.openclaw/config.yaml
gateway:
  port: 18789
  
tools:
  web:
    search:
      provider: brave
```

---

## Verified Integrations

These integrations have been verified and tested:

### GitHub Integration

**What it does:** Access GitHub repos, PRs, issues, CI status via the `gh` CLI

**Setup:**
```bash
# 1. Install gh CLI
brew install gh  # macOS
# or apt install gh  # Linux

# 2. Authenticate
git auth login

# 3. Set token in env (if using in container/headless)
export GITHUB_TOKEN="ghp_your_token"
```

**Use via chat:**
```
User: List my open PRs in openclaw/openclaw
Agent: [Uses gh pr list to show PRs]

User: Check the status of PR #42
Agent: [Uses gh pr view 42 --json state,checks]

User: Comment "LGTM" on PR #42
Agent: [Uses gh pr comment 42 --body "LGTM"]
```

---

### Calendar Integration (CalDAV)

**What it does:** Read/write calendar events via CalDAV protocol

**Setup:**
```bash
# 1. Install required binaries
pip install vdirsyncer
pip install khal

# 2. Configure vdirsyncer
# ~/.config/vdirsyncer/config
[pair my_calendar]
a = my_local
b = my_remote
collections = ["from a", "from b"]

[storage my_local]
type = filesystem
path = ~/.calendars/
fileext = .ics

[storage my_remote]
type = caldav
url = https://caldav.fastmail.com/dav/calendars/
username = your@email.com
password = your_app_password
```

**Use via chat:**
```
User: What's on my calendar today?
Agent: [Uses khal list today to show events]

User: Add a meeting tomorrow at 2pm with Alice
Agent: [Uses khal new to create event]
```

---

### Email Integration (IMAP/SMTP)

**What it does:** Read inbox and send emails

**Setup:**
```bash
# Set credentials in environment
export EMAIL_IMAP_SERVER="imap.gmail.com"
export EMAIL_IMAP_USER="you@gmail.com"
export EMAIL_IMAP_PASS="your_app_password"
export EMAIL_SMTP_SERVER="smtp.gmail.com"
export EMAIL_SMTP_USER="you@gmail.com"
export EMAIL_SMTP_PASS="your_app_password"
```

**Use via chat:**
```
User: Check my unread emails
Agent: [Connects to IMAP, fetches unread count]

User: Send an email to team@company.com about the release
Agent: [Drafts via chat, sends via SMTP]
```

---

### Web Search Integration

**What it does:** Search the web via Brave, Google, or other providers

**Setup:**
```bash
# For Brave Search
export BRAVE_API_KEY="your_brave_key"

# For Google Custom Search
export GOOGLE_SEARCH_API_KEY="your_key"
export GOOGLE_SEARCH_CX="your_cx"
```

**Configure in openclaw.json:**
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "brave",
        "apiKey": { "secretRef": "BRAVE_API_KEY" }
      }
    }
  }
}
```

**Use via chat:**
```
User: Search for OpenClaw tutorials
Agent: [Uses web_search to find results]

User: What are the latest AI agent frameworks?
Agent: [Searches, extracts, summarizes]
```

---

## Integration Configuration Patterns

### Pattern 1: Simple Environment Variable

For quick setup:

```bash
export API_KEY="your_key"
```

Then in chat:
```
User: Use my API key to check the weather
Agent: [Reads API_KEY from env, makes API call]
```

### Pattern 2: SecretRef with Fallback

For production:

```yaml
# config.yaml
integrations:
  github:
    token:
      secretRef: GITHUB_TOKEN
    # Fallback to direct value (not recommended)
    # token: "ghp_xxx"
```

### Pattern 3: Multiple Integrations

Configure several at once:

```yaml
# config.yaml
models:
  providers:
    anthropic:
      apiKey: { secretRef: ANTHROPIC_API_KEY }
    openai:
      apiKey: { secretRef: OPENAI_API_KEY }

integrations:
  github: { token: { secretRef: GITHUB_TOKEN } }
  calendar: { password: { secretRef: CALENDAR_PASSWORD } }
  email: { 
    imap: { user: "you@gmail.com", pass: { secretRef: EMAIL_PASSWORD } },
    smtp: { user: "you@gmail.com", pass: { secretRef: EMAIL_PASSWORD } }
  }
```

---

## Testing Your Integrations

### Test GitHub
```bash
gh auth status
gh repo list
```

### Test Calendar
```bash
khal list today
```

### Test Email
```bash
# Use openclaw to test
openclaw config get tools.email.imap.user
```

---

## Common Configuration Mistakes

| Mistake | Solution |
|---------|----------|
| Token in plain text config | Use `secretRef` instead |
| Wrong env var name | Check `openclaw config get <key>` |
| Missing `export` in .bashrc | Ensure vars are exported, not just set |
| API key without proper scopes | Verify token has required permissions |
| Credentials in git repo | Add to `.gitignore`, use env vars |

---

## Security Best Practices

1. **Never commit credentials** - Always use env vars or SecretRef
2. **Use app passwords** - For email/calendar, not your main password
3. **Rotate tokens regularly** - Set calendar reminders to rotate API keys
4. **Scope tokens narrowly** - GitHub tokens should have minimal permissions
5. **Check `openclaw security audit`** - Run regularly to find exposed credentials

---

## Related Modules and Next Steps

- Previous: [04-skills](../04-skills/) - How to install and use skills
- Next: [06-automation](../06-automation/) - Automating with cron
- Security: [09-security-and-ops](../09-security-and-ops/) - Production security
- Reference: `openclaw config --help`

---

**Time to complete:** 30 minutes  
**Prerequisites:** [04-skills](../04-skills/)  
**Outcome:** External services connected via secure configuration
