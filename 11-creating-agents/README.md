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
- You're building automation systems

## Prerequisites

**OpenClaw Version:** v0.31+ (for `openclaw agents add` wizard)

**Required:** A working main agent ([01-getting-started](../01-getting-started/))

## Quick Start

### Step 1: Create a New Agent

Use the agent wizard:

```bash
openclaw agents add work
```

This creates:
- Agent directory: `~/.openclaw/agents/work/`
- State directory: `~/.openclaw/agents/work/agent/`
- Session store: `~/.openclaw/agents/work/sessions/`
- Auth profiles: `~/.openclaw/agents/work/agent/auth-profiles.json`

### Step 2: Configure the Workspace

Create required configuration files in the workspace:

**AGENTS.md** - Agent identity and scope:
```markdown
# AGENTS.md - Work Agent

## Agent Identity
**Name:** WorkBot
**Purpose:** Handle work-related tasks and coding
**Task:** Daily PR reviews, code documentation

## Scope
- Monitor GitHub repositories
- Summarize code changes
- Post to work Slack channel

## Constraints
- Only accesses work repositories
- Never modifies personal files
```

**SOUL.md** - Core principles:
```markdown
# SOUL.md

## Core Identity
I am WorkBot, focused on development tasks and code review.

## Principles
- Thorough code analysis
- Clear documentation
- Security-conscious
```

**USER.md** - Who this agent helps:
```markdown
# USER.md

## User
**Name:** [Your name]
**Work hours:** 9 AM - 5 PM
**Channel:** Slack #engineering
```

**HEARTBEAT.md** - Periodic tasks:
```markdown
# HEARTBEAT.md

## Daily Tasks
- [ ] Check overnight PRs
- [ ] Summarize changes
- [ ] Post to Slack
```

**TOOLS.md** - Available integrations:
```markdown
# TOOLS.md

## GitHub
- Token: [via env var GITHUB_TOKEN_WORK]
- Repos: company/backend, company/frontend

## Slack
- Channel: #engineering
- Token: [via env var SLACK_TOKEN_WORK]
```

### Step 3: Set Up Channel Bindings

Route specific channels to this agent:

```bash
# Bind work Slack to the work agent
openclaw agents bind --agent work --bind slack:engineering

# Bind work GitHub to the work agent
openclaw agents bind --agent work --bind github:company

# Verify bindings
openclaw agents list --bindings
```

### Step 4: Configure Channel Credentials (IMPORTANT)

**Binding routes incoming messages to the agent, but each agent needs its own credentials to respond.**

For **Discord** and other channels, create an `auth-profiles.json` file:

```bash
# Create the auth profiles directory
mkdir -p ~/.openclaw/agents/work/agent

# Create the auth file
cat > ~/.openclaw/agents/work/agent/auth-profiles.json << 'EOF'
{
  "discord": {
    "default": {
      "botToken": "YOUR_WORK_BOT_TOKEN_HERE"
    }
  },
  "slack": {
    "default": {
      "botToken": "xoxb-YOUR-WORK-SLACK-TOKEN"
    }
  }
}
EOF
```

**⚠️ Critical:** Each agent needs its **own Discord bot** (separate application at https://discord.com/developers/applications). Sharing tokens between agents causes routing confusion.

**Why this is needed:**
- Binding (`--bind`) = Routes **incoming** messages to the agent
- Auth profiles = Allows agent to send **outgoing** responses
- Without auth profiles, the agent cannot reply to the channel

### Step 5: Configure Identity

Set the agent's name, avatar, and theme:

```bash
openclaw agents set-identity --agent work \
  --name "WorkBot" \
  --emoji "💼" \
  --avatar avatars/workbot.png
```

Or load from IDENTITY.md:
```bash
openclaw agents set-identity --agent work --from-identity
```

### Step 6: Restart and Verify

```bash
# Restart gateway to apply changes
openclaw gateway restart

# Verify agent is running
openclaw agents list --bindings

# Check channel status
openclaw channels status --probe
```

## Multi-Agent Architecture

```
Main Agent (default)
├── Workspace: ~/.openclaw/workspace/
├── Agent ID: main
├── State: ~/.openclaw/agents/main/
└── General purpose

Work Agent
├── Workspace: ~/.openclaw/workspace-work/
├── Agent ID: work
├── State: ~/.openclaw/agents/work/
├── Slack: #engineering
├── GitHub: company repos
└── Specialized: Code reviews

Personal Agent
├── Workspace: ~/.openclaw/workspace-personal/
├── Agent ID: personal
├── State: ~/.openclaw/agents/personal/
├── Telegram: @personalbot
└── Specialized: Life admin
```

## Directory Structure Explained

```
~/.openclaw/
├── openclaw.json              # Main configuration
├── agents/
│   ├── main/
│   │   ├── agent/             # State and auth
│   │   │   └── auth-profiles.json
│   │   └── sessions/          # Chat history
│   ├── work/
│   │   ├── agent/
│   │   │   └── auth-profiles.json
│   │   └── sessions/
│   └── personal/
│       ├── agent/
│       └── sessions/
├── workspace/                 # Default workspace (main agent)
├── workspace-work/           # Work agent workspace
└── workspace-personal/      # Personal agent workspace
```

**Important Rules:**
- **Never reuse agentDir** across agents (causes auth/session collisions)
- Each agent has **separate auth profiles** and **isolated sessions**
- Workspaces are the **default cwd**, not hard sandboxes

## Communication Between Agents

### Pattern 1: File-Based (Shared Memory)

Agents read/write shared files:

```
~/.openclaw/shared/
├── work-tasks.md      # Work agent writes, main agent reads
├── research-notes.md  # Research agent writes, all agents read
└── daily-report.md    # Aggregated daily summaries
```

### Pattern 2: Message-Based

Main agent triggers others via natural language:

```
User: Ask work agent to review PRs
Main Agent: [Messages work agent's channel]
Work Agent: [Processes and responds directly]
```

### Pattern 3: Cron-Based (Scheduled)

Coordinate with time offsets:

```bash
# 8:00 AM - Work agent checks PRs
openclaw cron add \
  --name "work-check-prs" \
  --cron "0 8 * * 1-5" \
  --agent work \
  --message "Check overnight PRs and summarize"

# 8:05 AM - Main agent aggregates
openclaw cron add \
  --name "main-digest" \
  --cron "5 8 * * 1-5" \
  --message "Read work-tasks.md and post daily summary"
```

## Real-World Examples

### Example 1: Solo Founder Team (4 agents)

**Trebuh's setup** (verified in community):

| Agent | Role | Tasks | Channel |
|-------|------|-------|---------|
| Milo | Strategy Lead | Daily standups, priority setting | Telegram @milo |
| Josh | Business | Metrics, competitor monitoring | Telegram @josh |
| Marketing | Creative | Content ideas, trend research | Telegram @marketing |
| Dev | Technical | CI/CD monitoring, PR reviews | Telegram @dev |

**Shared structure:**
```
team/
├── GOALS.md              # All agents read (OKRs)
├── DECISIONS.md          # Append-only decisions
├── PROJECT_STATUS.md     # Current state
└── agents/
    ├── milo/            # Private notes
    ├── josh/
    ├── marketing/
    └── dev/
```

**Telegram routing:**
- `@milo` → Strategy agent
- `@josh` → Business agent
- `@marketing` → Marketing agent
- `@dev` → Dev agent

### Example 2: Content Factory (3 agents)

**Setup from awesome-openclaw-usecases:**

| Agent | Channel | Task | Schedule |
|-------|---------|------|----------|
| Research | Discord #research | Find trending topics | 8 AM daily |
| Writing | Discord #scripts | Write drafts | 9 AM daily |
| Design | Discord #thumbnails | Generate images | 10 AM daily |

**Workflow:**
1. Research posts top 5 opportunities in #research
2. Writing takes best idea, writes script in #scripts
3. Design generates thumbnail in #thumbnails

## Best Practices

### 1. Workspace Isolation

Create separate directories:
```bash
mkdir -p ~/.openclaw/workspace-{work,personal,research}
```

### 2. Separate Auth Profiles

Each agent has its own credentials:
```bash
# Work agent uses work GitHub token
export GITHUB_TOKEN_WORK="ghp_work_token"

# Personal agent uses personal token
export GITHUB_TOKEN_PERSONAL="ghp_personal_token"
```

### 3. Skill Scoping

Each workspace gets its own `skills/` folder:
```
workspace-work/skills/
├── github-work/       # Work GitHub integration
└── slack-work/        # Work Slack integration

workspace-personal/skills/
├── telegram-personal/ # Personal Telegram
└── calendar-personal/ # Personal calendar
```

### 4. Clear Boundaries

Document what each agent does:
```markdown
# AGENTS.md

## This Agent DOES:
- Monitor work GitHub repos
- Post to #engineering Slack
- Summarize code changes

## This Agent DOES NOT:
- Access personal repositories
- Send emails
- Modify main agent files
```

## Common Mistakes

| Mistake | Why It's Bad | Solution |
|---------|--------------|----------|
| Reusing agentDir | Auth/session collisions | Use `openclaw agents add` for each |
| Sharing workspaces | Memory bleed | Separate directories |
| **Same channel tokens** | **Cross-routing chaos** | **One token per agent per channel** |
| **Missing auth-profiles.json** | **Agent can't respond** | **Create `~/.openclaw/agents/<name>/agent/auth-profiles.json`** |
| No bindings | All agents receive all messages | Explicit `agents bind` |
| Unclear scope | Agents overlap | Document in AGENTS.md |

## Troubleshooting: Agent Not Responding in Channel

**Problem:** You bound the agent to Discord, but the main agent still responds (or no response at all).

**Diagnosis Steps:**

**Step 1: Check the binding exists**
```bash
openclaw agents list --bindings
# Should show: docloop → discord:YOUR_CHANNEL_ID
```

**Step 2: Check auth-profiles.json exists**
```bash
ls ~/.openclaw/agents/docloop/agent/auth-profiles.json
# If missing, the agent can't respond!
```

**Step 3: Create auth-profiles.json**
```bash
mkdir -p ~/.openclaw/agents/docloop/agent
cat > ~/.openclaw/agents/docloop/agent/auth-profiles.json << 'EOF'
{
  "discord": {
    "default": {
      "botToken": "YOUR_BOT_TOKEN_HERE"
    }
  }
}
EOF
```

**Step 4: Get a separate Discord bot token**
- Go to https://discord.com/developers/applications
- Create **New Application** (don't reuse existing bot)
- Bot → Add Bot → Copy Token
- Enable **Message Content Intent**

**Step 5: Restart gateway**
```bash
openclaw gateway restart
```

**Why this happens:**
- **Binding** (`--bind`) = Routes **incoming** messages to agent ✅
- **Auth profiles** = Allows **outgoing** responses ❌ (often forgotten!)
- Each agent needs its **own bot token** to send messages

## CLI Reference

```bash
# Create and manage agents
openclaw agents add <name>                    # Create new agent
openclaw agents list                          # List all agents
openclaw agents list --bindings              # List with channel bindings
openclaw agents delete <name>                 # Remove agent

# Configure identity
openclaw agents set-identity --agent <name> \
  --name "Display Name" \
  --emoji "💼" \
  --avatar avatars/bot.png

# Channel routing
openclaw agents bind --agent <name> --bind <channel:account>
openclaw agents unbind --agent <name> --bind <channel:account>
openclaw agents bindings                      # Show all bindings

# Verify
openclaw channels status --probe              # Check channel health
```

## Advanced: One WhatsApp, Multiple People

Route different WhatsApp DMs to different agents:

```json
{
  "agents": {
    "list": [
      { "id": "alex", "workspace": "~/.openclaw/workspace-alex" },
      { "id": "mia", "workspace": "~/.openclaw/workspace-mia" }
    ]
  },
  "bindings": [
    {
      "agentId": "alex",
      "match": { "channel": "whatsapp", "peer": { "kind": "direct", "id": "+15551230001" } }
    },
    {
      "agentId": "mia",
      "match": { "channel": "whatsapp", "peer": { "kind": "direct", "id": "+15551230002" } }
    }
  ],
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551230001", "+15551230002"]
    }
  }
}
```

## Related Modules

- Previous: [10-cli-and-reference](../10-cli-and-reference/)
- Security: [09-security-and-ops](../09-security-and-ops/)
- Automation: [06-automation](../06-automation/)
- Integration: [05-integrations](../05-integrations/)

---

**Time to complete:** 45 minutes  
**Prerequisites:** [01-getting-started](../01-getting-started/), OpenClaw v0.31+  
**Outcome:** Multiple specialized agents with isolated workspaces and coordinated workflows

---

## References

- Official docs: https://docs.openclaw.ai/cli/agents
- Multi-agent routing: https://docs.openclaw.ai/concepts/multi-agent
- Community example: https://github.com/hesamsheikh/awesome-openclaw-usecases (multi-agent-team.md)
- Real-world setup: Run Lobster multi-agent guide