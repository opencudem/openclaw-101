# 03 - Memory

> Make OpenClaw remember who you are, what you like, and what you're working on.

## What this module solves

Without memory, every conversation starts fresh. With memory, your assistant knows your preferences, past conversations, and ongoing projects — making it genuinely useful.

## When to use this module

- You want personalized responses
- You're working on long-term projects
- You want the assistant to learn your style

## Types of Memory

OpenClaw has multiple memory layers:

```
┌─────────────────────────────────────────┐
│           Conversation Context          │  ← Current chat (temporary)
├─────────────────────────────────────────┤
│         File-Based Memory               │  ← ~/.openclaw/memory/
├─────────────────────────────────────────┤
│         Skill Memory                    │  ← Skills store data
├─────────────────────────────────────────┤
│         External Integrations           │  ← Notion, databases
└─────────────────────────────────────────┘
```

## Quick Start

### File-Based Memory

Create memory files that the assistant can read:

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

## Memory in Practice

### Mention memory in chat

```
User: Remember that I prefer brief responses
Agent: Got it — I'll keep my replies concise.

[This gets added to your memory file automatically]
```

### Query your memory

```
User: What did we discuss yesterday?
Agent: [Reads from daily notes and summarizes]
```

### Update preferences

```
User: Actually, I changed my mind about the database. Use SQLite instead.
Agent: Updated. I've noted the switch to SQLite in your project memory.
```

## Copy-paste examples

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

### Auto-remember script

Add to your OpenClaw skills:

```javascript
// ~/.openclaw/skills/memory-helper/scripts/save-note.js
const fs = require('fs');
const path = require('path');

function saveNote(category, content) {
  const date = new Date().toISOString().split('T')[0];
  const notePath = path.join(
    process.env.HOME,
    '.openclaw/memory',
    category,
    `${date}.md`
  );
  
  fs.mkdirSync(path.dirname(notePath), { recursive: true });
  fs.appendFileSync(notePath, `\n## ${new Date().toLocaleTimeString()}\n${content}\n`);
  
  return `Saved to ${category}/${date}.md`;
}

module.exports = { saveNote };
```

## Common mistakes and troubleshooting

| Problem | Solution |
|---------|----------|
| Memory not being read | Check file location: must be in ~/.openclaw/memory/ |
| Too much old context | Archive old daily notes; keep only recent ones |
| Sensitive info exposed | Use restricted channels; separate personal/work memory |

## Advanced patterns

### Structured memory with JSON

For programmatic access:

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

### Memory search

Create a simple search:

```bash
# Search memory files
grep -r "database" ~/.openclaw/memory/

# Or use a skill that indexes memory
openclaw skill search-memory "database decision"
```

## Related modules and next step

- Previous: [02-channels](../02-channels/)
- Next: [04-skills](../04-skills/) — Extend capabilities with skills
- Related: [Workflows with memory](../08-workflows/README.md)

---

**Time to complete:** 30 minutes  
**Prerequisites:** [02-channels](../02-channels/)  
**Outcome:** Personalized assistant that remembers context
