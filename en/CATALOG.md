# Catalog

> Curated skills and integrations for OpenClaw

## Skills by Category

### 🌤️ Information

| Skill | Description | Install |
|-------|-------------|---------|
| weather | Get weather and forecasts | `openclaw skill install weather` |
| news | Fetch news headlines | `openclaw skill install news` |
| stocks | Stock price tracking | `openclaw skill install stocks` |
| crypto | Cryptocurrency prices | `openclaw skill install crypto` |

### 📅 Productivity

| Skill | Description | Install |
|-------|-------------|---------|
| calendar | Calendar management | `openclaw skill install calendar` |
| todo | Task management | `openclaw skill install todo` |
| reminders | Set reminders | `openclaw skill install reminders` |
| notes | Note taking | `openclaw skill install notes` |

### 💬 Communication

| Skill | Description | Install |
|-------|-------------|---------|
| email | Send and receive email | `openclaw skill install email` |
| slack | Enhanced Slack features | `openclaw skill install slack` |
| sms | Send SMS messages | `openclaw skill install sms` |

### 🔧 Development

| Skill | Description | Install |
|-------|-------------|---------|
| github | GitHub integration | `openclaw skill install github` |
| git | Git operations | `openclaw skill install git` |
| docker | Docker management | `openclaw skill install docker` |
| npm | Node.js package management | `openclaw skill install npm` |

### 🌐 Web

| Skill | Description | Install |
|-------|-------------|---------|
| browser-automation | Web automation | `openclaw skill install browser-automation` |
| search | Web search | `openclaw skill install search` |
| scraper | Data extraction | `openclaw skill install scraper` |

### 🗄️ Data

| Skill | Description | Install |
|-------|-------------|---------|
| postgres | PostgreSQL database | `openclaw skill install postgres` |
| sqlite | SQLite database | `openclaw skill install sqlite` |
| redis | Redis operations | `openclaw skill install redis` |
| notion | Notion integration | `openclaw skill install notion` |

### 🎯 Project Management

| Skill | Description | Install |
|-------|-------------|---------|
| linear | Linear issue tracking | `openclaw skill install linear` |
| trello | Trello boards | `openclaw skill install trello` |
| asana | Asana tasks | `openclaw skill install asana` |
| jira | Jira integration | `openclaw skill install jira` |

### 🤖 AI & ML

| Skill | Description | Install |
|-------|-------------|---------|
| image-gen | Generate images | `openclaw skill install image-gen` |
| speech | Text-to-speech | `openclaw skill install speech` |
| summarizer | Content summarization | `openclaw skill install summarizer` |
| translator | Language translation | `openclaw skill install translator` |

## Recommended Bundles

### Starter Pack

```bash
# Essential skills for beginners
openclaw skill install weather
openclaw skill install calendar
openclaw skill install todo
openclaw skill install reminders
```

### Developer Pack

```bash
# Skills for software development
openclaw skill install github
openclaw skill install git
openclaw skill install docker
openclaw skill install postgres
```

### Automation Pack

```bash
# Skills for workflow automation
openclaw skill install browser-automation
openclaw skill install search
openclaw skill install email
openclaw skill install notion
```

## Custom Skills

Create your own skills in `~/.openclaw/skills/`:

```
my-skill/
├── SKILL.md          # Documentation
├── scripts/          # Executable scripts
│   └── my-tool.js
└── references/       # Reference files
    └── api-docs.md
```

See [04-skills](04-skills/) for the full guide.

## Submit to ClawHub

Share your skills with the community:

1. Publish to npm: `npm publish`
2. Submit to [ClawHub](https://clawhub.ai)
3. Tag appropriately for discovery

---

Last updated: 2026-03-31
