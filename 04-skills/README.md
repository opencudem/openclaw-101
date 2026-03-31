# 04 - Skills

> Teach OpenClaw new tricks: install, create, and manage skills from ClawHub and beyond.

## What this module solves

Skills are how OpenClaw learns to do new things. This module covers finding skills from ClawHub, understanding how they work, and creating your own.

## When to use this module

- You want to extend OpenClaw's capabilities
- You need custom integrations
- You want to automate specific tasks

## Skill Architecture

```
Skill Sources (in priority order):
├── 1. Bundled Skills    (comes with OpenClaw)
├── 2. Workspace Skills  (~/.openclaw/skills/)
├── 3. ClawHub Skills    (installed via openclaw skill install)
└── 4. Custom Entries    (config-specified paths)
```

Later sources override earlier ones. This lets you customize bundled skills.

## Quick Start

### Finding Skills

Browse available skills:

```bash
# Search ClawHub
openclaw skill search weather

# List installed skills
openclaw skill list

# Get skill info
openclaw skill info weather
```

### Installing Skills

```bash
# Install from ClawHub
openclaw skill install weather

# Install specific version
openclaw skill install weather@1.2.0

# Update a skill
openclaw skill update weather

# Remove a skill
openclaw skill remove weather
```

### Installing via npx (for development)

```bash
# Quick install without adding to ClawHub
npx openclaw-skill-weather
```

## Skill Structure

A skill is a folder with this structure:

```
my-skill/
├── SKILL.md          # Documentation and spec
├── package.json      # Dependencies (optional)
├── references/       # Reference files
│   └── api-docs.md
└── scripts/          # Executable scripts
    ├── get-weather.js
    └── forecast.js
```

### SKILL.md Format

```markdown
# My Skill

## Description
What this skill does and when to use it.

## Tools

### get-weather
Get current weather for a location.

**Parameters:**
- location (string, required): City name

**Example:**
```javascript
getWeather({ location: "New York" })
```
```

## Creating Your First Skill

```bash
# Create skill directory
mkdir -p ~/.openclaw/skills/my-first-skill
cd ~/.openclaw/skills/my-first-skill

# Create SKILL.md
cat > SKILL.md << 'EOF'
# Hello World Skill

## Description
A simple greeting skill for testing.

## Tools

### greet
Say hello to someone.

**Parameters:**
- name (string, required): Person to greet

**Example:**
```javascript
greet({ name: "World" })
// Returns: "Hello, World!"
```
EOF

# Create script
mkdir -p scripts
cat > scripts/greet.js << 'EOF'
module.exports = async function greet({ name }) {
  return `Hello, ${name}!`;
};
EOF
```

Test it:

```
User: Greet Alice
Agent: [Uses greet skill] Hello, Alice!
```

## Copy-paste examples

### Weather skill wrapper

```javascript
// ~/.openclaw/skills/local-weather/scripts/weather.js
const fetch = require('node-fetch');

async function getWeather({ location }) {
  const response = await fetch(
    `https://wttr.in/${location}?format=j1`
  );
  const data = await response.json();
  
  return {
    temp: data.current_condition[0].temp_C,
    condition: data.current_condition[0].weatherDesc[0].value,
    humidity: data.current_condition[0].humidity
  };
}

module.exports = { getWeather };
```

### Git helper skill

```javascript
// ~/.openclaw/skills/git-helper/scripts/repo.js
const { execSync } = require('child_process');

module.exports = {
  getStatus: () => {
    return execSync('git status --short', { encoding: 'utf8' });
  },
  
  getLastCommit: () => {
    return execSync('git log -1 --oneline', { encoding: 'utf8' });
  },
  
  getBranch: () => {
    return execSync('git branch --show-current', { encoding: 'utf8' }).trim();
  }
};
```

## Common mistakes and troubleshooting

| Problem | Solution |
|---------|----------|
| Skill not appearing | Check SKILL.md syntax; verify folder structure |
| Script errors | Check file permissions; ensure shebang line present |
| Dependencies missing | Run `npm install` in skill directory |
| Skill not found | Verify skill is in search path; restart gateway |

## Advanced patterns

### Skill dependencies

```json
// package.json in skill folder
{
  "name": "my-skill",
  "dependencies": {
    "axios": "^1.0.0",
    "cheerio": "^1.0.0"
  }
}
```

Then run `npm install` in the skill directory.

### Conditional skills

Load skills only in specific contexts:

```yaml
# ~/.openclaw/config.yaml
skills:
  entries:
    work-git:
      path: ./skills/work-git
      condition: "channel.type === 'slack'"
```

### Skill composition

Skills can use other skills:

```javascript
// In your skill script
const weather = await skills.weather.getWeather({ location });
const calendar = await skills.calendar.getToday();

return `It's ${weather.temp}°C. You have ${calendar.events.length} events today.`;
```

## Related modules and next step

- Previous: [03-memory](../03-memory/)
- Next: [05-integrations](../05-integrations/)
- Reference: [Skills Config](https://docs.openclaw.ai/tools/skills-config)
- Browse: [ClawHub](https://clawhub.ai)

---

**Time to complete:** 60 minutes  
**Prerequisites:** [03-memory](../03-memory/)  
**Outcome:** Custom skills installed and created, extended assistant capabilities
