# 11 - Creating Additional Agents

> Create specialized agents with their own workspaces for specific tasks.

## What this module solves

Once you have your main OpenClaw agent running, you may want additional agents for specific purposes:
- A **documentation agent** that maintains your wiki
- A **research agent** that monitors sources and reports findings
- A **coding agent** that handles specific projects
- A **personal assistant** for different contexts (work vs. personal)

Each agent gets its own workspace, configuration, and memory - completely isolated from your main agent.

## When to use this module

- You want specialized agents for different tasks
- You need isolated contexts (work vs. personal)
- You want to experiment without affecting your main setup
- You're building automation systems (like DocLoop)

## Quick Start

### Step 1: Create a New Agent Workspace

```bash
# Create a dedicated directory for your new agent
mkdir -p ~/.openclaw/workspace/my-second-agent
cd ~/.openclaw/workspace/my-second-agent

# Initialize git (optional but recommended)
git init
```

### Step 2: Create Required Configuration Files

Every agent workspace needs these files:

**AGENTS.md** - Agent identity and scope:
```markdown
# AGENTS.md - My Second Agent

## Agent Identity
**Name:** ResearchBot
**Purpose:** Monitor tech news and summarize findings
**Task:** Daily research reports

## Scope
- Monitor specific RSS feeds and GitHub repos
- Summarize findings
- Report to Discord channel

## Constraints
- Only reads from specified sources
- Never modifies main agent workspace
```

**SOUL.md** - Core principles:
```markdown
# SOUL.md

## Core Identity
I am ResearchBot, focused on information gathering and summarization.

## Principles
- Evidence-based reporting
- Clear summaries
- No speculation
```

**USER.md** - Who this agent helps:
```markdown
# USER.md

## User
**Name:** [Your name]
**Preferences:** Daily summaries at 9 AM
**Channel:** Discord #research
```

**HEARTBEAT.md** - Periodic tasks:
```markdown
# HEARTBEAT.md

## Daily Tasks
- [ ] Check RSS feeds
- [ ] Summarize new findings
- [ ] Post to Discord
```

**TOOLS.md** - Available tools and credentials:
```markdown
# TOOLS.md

## RSS Feeds
- https://techcrunch.com/feed
- https://news.ycombinator.com/rss

## Discord
- Channel: #research
- Webhook: [configured]
```

### Step 3: Configure the Agent

Create `~/.openclaw/agents/researchbot.json`:

```json
{
  "name": "researchbot",
  "workspace": "~/.openclaw/workspace/my-second-agent",
  "heartbeat": {
    "every": "1h",
    "target": "discord",
    "to": "YOUR_CHANNEL_ID"
  },
  "skills": {
    "enabled": ["web-search", "rss-reader"]
  }
}
```

### Step 4: Start the Agent

```bash
# Option A: Run directly
openclaw agents run researchbot

# Option B: Add to your main agent's crontab
openclaw cron add \
  --name "researchbot-daily" \
  --cron "0 9 * * *" \
  --message "Run researchbot daily scan" \
  --session isolated \
  --agent researchbot
```

## Multi-Agent Architecture

```
Main Agent (you)
├── Workspace: ~/.openclaw/workspace/
├── General purpose
└── Controls other agents

ResearchBot
├── Workspace: ~/.openclaw/workspace/researchbot/
├── Specialized: News monitoring
└── Reports to main agent

DocLoop
├── Workspace: ~/.openclaw/workspace/docloop/
├── Specialized: Documentation
└── Creates PRs, reports status
```

## Communication Between Agents

### Pattern 1: File-Based
Agents write to shared files:
```
~/.openclaw/shared/
├── research-findings.md
├── todo-list.md
└── daily-report.md
```

### Pattern 2: Message-Based
Main agent triggers others via messages:
```
User: Ask researchbot to check for updates
Main Agent: [Sends message to researchbot channel]
ResearchBot: [Processes and responds]
```

### Pattern 3: Cron-Based
Scheduled coordination:
```bash
# 8:00 AM - ResearchBot gathers data
# 8:05 AM - Main Agent processes findings
# 8:10 AM - Summary posted to Discord
```

## Best Practices

### 1. Workspace Isolation
```bash
# Each agent has its own directory
~/.openclaw/workspace/
├── main/              # Your primary agent
├── researchbot/       # Research agent
├── docloop/          # Documentation agent
└── coding-assistant/ # Coding helper
```

### 2. Separate Memory
Each agent maintains its own:
- `MEMORY.md` (long-term)
- `memory/YYYY-MM-DD.md` (daily logs)
- `AGENTS.md` (identity)

### 3. Skill Scoping
```yaml
# researchbot/.openclaw/skills/
skills:
  entries:
    rss-reader:        # Only RSS-related
    web-search:        # Web research
    # No git, no email, no browser automation
```

### 4. Clear Boundaries
```markdown
# AGENTS.md - Boundaries section
## This Agent Does:
- Read RSS feeds
- Summarize articles
- Post to Discord

## This Agent Does NOT:
- Modify git repos
- Send emails
- Access main agent memory
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Sharing workspaces | Always create separate directories |
| Mixing memory | Each agent has its own memory files |
| Unclear scope | Document boundaries in AGENTS.md |
| No heartbeat | Set up periodic check-ins |
| Skill overlap | Scope skills to agent purpose |

## Example: DocLoop Agent Structure

```
~/.openclaw/workspace/docloop-agent/
├── AGENTS.md          # DocLoop identity
├── SOUL.md            # Documentation principles
├── USER.md            # Owner preferences
├── HEARTBEAT.md       # 6-phase workflow
├── TOOLS.md           # Exa API, GitHub token
├── current_run.md     # Phase tracking
├── RESEARCH.md        # Findings cache
└── .git/              # Version control
```

## Related Modules

- Previous: [10-cli-and-reference](../10-cli-and-reference/)
- Security: [09-security-and-ops](../09-security-and-ops/)
- Automation: [06-automation](../06-automation/)

---

**Time to complete:** 45 minutes  
**Prerequisites:** [01-getting-started](../01-getting-started/)  
**Outcome:** Multiple specialized agents working together
