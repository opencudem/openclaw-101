# 03 - Memory

> Make OpenClaw remember who you are, what you like, and what you're working on.

## What this module solves

Without memory, every conversation starts fresh. With memory, your assistant knows your preferences, past conversations, and ongoing projects — making it genuinely useful.

## When to use this module

- You want personalized responses
- You're working on long-term projects
- You want the assistant to learn your style

## Memory Locations

OpenClaw reads memory from these locations (in priority order):

| Location | Purpose | Priority |
|----------|---------|----------|
| `MEMORY.md` | Long-term curated memory | Highest |
| `memory/*.md` | Daily notes, project context | High |
| `~/.openclaw/memory/` | Global cross-workspace memory | Medium |

## Memory CLI Commands

OpenClaw provides CLI commands to manage and search memory:

```bash
# Check memory status
openclaw memory status

# Index memory files (refresh search index)
openclaw memory index

# Search your memory
openclaw memory search "database decision"
openclaw memory search --query "what did we decide about postgres"
```

## Quick Start

### 1. Check Memory Status

```bash
openclaw memory status
```

This shows which memory files are loaded and their status.

### 2. Create Memory Files

Create memory files that OpenClaw automatically reads:

```bash
# Create memory directory
mkdir -p ~/.openclaw/memory

# Add a memory file
cat > ~/.openclaw/memory/about-me.md << 'EOF'
# About Me

- Name: [Your name]
- Location: [Your location]
- Work: [What you do]
- Preferences:
  - Communication style: [direct/detailed/friendly]
  - Hours: [when you work]
  - Timezone: [your timezone]
EOF
```

The assistant automatically reads these files when starting a conversation.

### 3. Index and Search

```bash
# Index your memory files
openclaw memory index

# Search for specific information
openclaw memory search "your query here"
```

## Memory Patterns

### Daily Notes Pattern

Create dated notes for ongoing context:

```bash
# Create today's note
mkdir -p ~/.openclaw/memory/daily
date +%Y-%m-%d.md

# Structure:
# ## 2026-03-31
# ### Priorities
# - Launch new feature
# - Review PRs
# 
# ### Decisions
# - Chose PostgreSQL over MongoDB
#
# ### Notes
# - Remember to ask about API limits
```

### Project Memory

Keep project-specific context:

```bash
mkdir -p ~/.openclaw/memory/projects

# ~/.openclaw/memory/projects/openclaw-101.md
# ## OpenClaw-101 Project
# 
# ### Goals
# - Create beginner-friendly guide
# - 10 learning modules
# 
# ### Tech Stack
# - Markdown
# - GitHub Pages
# 
# ### Current Status
# - Phase 1: Core modules (in progress)
```

### Workspace Memory

For project-specific memory, use the workspace root:

```bash
# In your project workspace
cat > MEMORY.md << 'EOF'
# Project Context

## Current Goal
Implement feature X by Friday

## Key Decisions
- Using Redis for caching
- API rate limit: 100 req/min

## Team
- Lead: Jane
- Backend: Bob
EOF
```

## Memory in Practice

### Mention memory in chat

```
User: Remember that I prefer brief responses
Agent: Got it — I'll keep my replies concise.

[This can be saved to your memory file]
```

### Query your memory

```bash
# Search for past decisions
openclaw memory search "database decision"

# The agent can also search during conversations
```

### Update preferences

```
User: Actually, I changed my mind about the database. Use SQLite instead.
Agent: Updated. I've noted the switch to SQLite in your project memory.
```

## Copy-Paste Examples

### Memory file template

```markdown
# User Profile

## Personal
- Name: 
- Role:
- Timezone:

## Communication Style
- Response length: [brief/detailed]
- Tone: [professional/casual]
- Format preference: [bullets/paragraphs]

## Current Projects
- [Project name]: [brief description]

## Useful Context
- [Anything the agent should know]
```

### Search memory

```bash
# Search for specific topics in your memory
openclaw memory search "postgres"

# Search with natural language
openclaw memory search --query "what database did we choose"
```

## Common Mistakes and Troubleshooting

| Problem | Solution |
|---------|----------|
| Memory not being read | Check file location: must be in `MEMORY.md`, `memory/*.md`, or `~/.openclaw/memory/` |
| Search returns nothing | Run `openclaw memory index` to refresh the index |
| Too much old context | Archive old daily notes; keep only recent ones |
| Sensitive info exposed | Use restricted channels; separate personal/work memory |

## Advanced Patterns

### Structured memory with JSON

For programmatic access, you can also use JSON:

```json
{
  "user": {
    "name": "Alex",
    "preferences": {
      "notifications": "summarized",
      "timezone": "America/New_York"
    }
  },
  "projects": {
    "openclaw-101": {
      "status": "active",
      "priority": "high"
    }
  }
}
```

### Automating memory indexing

Add to your shell profile to auto-index on login:

```bash
# ~/.bashrc or ~/.zshrc
alias om="openclaw memory"
alias oms="openclaw memory search"
```

## Related modules and next step

- Previous: [02-channels](../02-channels/)
- Next: [04-skills](../04-skills/) — Extend capabilities with skills
- Related: [Workflows with memory](../08-workflows/README.md)

---

**Time to complete:** 30 minutes  
**Prerequisites:** [02-channels](../02-channels/)  
**Outcome:** Personalized assistant that remembers context
