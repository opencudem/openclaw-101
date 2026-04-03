# Use Cases

> Map your goal to the right OpenClaw modules

## Personal Productivity

| Goal | Modules | Workflow |
|------|---------|----------|
| **Morning briefings** | 02-channels, 03-memory, 06-automation | [Morning Briefing](examples/personal-productivity/morning-briefing.md) |
| **Task management** | 03-memory, 04-skills | Custom todo skill |
| **Reminder system** | 06-automation | Cron-based reminders |
| **Note taking** | 03-memory, 02-channels | Daily notes pattern |
| **Research assistant** | 07-browser-automation | [Research Agent](examples/research-and-content/research-agent.md) |
| **Travel planning** | 07-browser-automation, 04-skills | Web research + calendar |

## Engineering / Development

| Goal | Modules | Workflow |
|------|---------|----------|
| **PR review alerts** | 02-channels, 05-integrations | [Engineering Inbox](examples/engineering/pr-alerts.md) |
| **Daily standups** | 06-automation, 05-integrations | Auto-generated summaries |
| **Incident response** | 02-channels, 09-security | Alert + approval pattern |
| **CI/CD monitoring** | 05-integrations, 06-automation | Status checks |
| **Code review** | 04-skills | GitHub integration |
| **Documentation** | 07-browser-automation, 03-memory | Auto-generate docs |

## Founder / Operations

| Goal | Modules | Workflow |
|------|---------|----------|
| **Email triage** | 05-integrations, 03-memory | Sort + summarize + respond |
| **Calendar coordination** | 05-integrations | Schedule optimization |
| **Follow-up system** | 03-memory, 06-automation | [Follow-up System](examples/founder-ops/follow-up-system.md) |
| **Weekly briefings** | 06-automation | Auto-generated reports |
| **Team updates** | 02-channels, 06-automation | Scheduled broadcasts |
| **Investor updates** | 08-workflows | Multi-source report |

## Content / Research

| Goal | Modules | Workflow |
|------|---------|----------|
| **Web research** | 07-browser-automation | [Research Agent](examples/research-and-content/research-agent.md) |
| **Content curation** | 07-browser-automation, 03-memory | Auto-collect + summarize |
| **Social media** | 05-integrations | Scheduled posts |
| **News monitoring** | 06-automation | Daily news digest |
| **Competitor tracking** | 07-browser-automation | Price/feature monitoring |

## Home / Lifestyle

| Goal | Modules | Workflow |
|------|---------|----------|
| **Smart home** | 05-integrations | IoT device control |
| **Shopping lists** | 03-memory | Shared family lists |
| **Meal planning** | 04-skills | Recipe + shopping integration |
| **Fitness tracking** | 05-integrations | Workout + health data |
| **Travel alerts** | 07-browser-automation | Price drops, availability |

## Quick Start by Role

### Developer

1. [01-getting-started](01-getting-started/) - Install
2. [02-channels](02-channels/) - Connect Slack/Discord
3. [04-skills](04-skills/) - Install GitHub skill
4. [06-automation](06-automation/) - Set up PR alerts

### Founder

1. [01-getting-started](01-getting-started/) - Install
2. [02-channels](02-channels/) - Connect Telegram/WhatsApp
3. [03-memory](03-memory/) - Set up memory
4. [08-workflows](08-workflows/) - Email triage workflow

### Researcher

1. [01-getting-started](01-getting-started/) - Install
2. [02-channels](02-channels/) - Connect preferred channel
3. [07-browser-automation](07-browser-automation/) - Browser skills
4. [08-workflows](08-workflows/) - Research workflow

## Decision Tree

```
I want to...
│
├─→ Automate a recurring task → [06-automation]
│   └─→ With web data → [07-browser-automation]
│
├─→ Connect to my tools → [05-integrations]
│   └─→ GitHub, Calendar, Email, etc.
│
├─→ Build custom functionality → [04-skills]
│   └─→ Browser automation → [07-browser-automation]
│
├─→ Set up for my team → [02-channels] + [09-security]
│
└─→ Just getting started → [01-getting-started]
    └─→ Then [02-channels] for your chat app
```

## Contribute Your Use Case

Have a unique use case? Share it!

1. Create a workflow in `examples/`
2. Document the modules used
3. Submit a PR

---

Last updated: 2026-03-31
