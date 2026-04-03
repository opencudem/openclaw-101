# Master OpenClaw in a Weekend

> The practical guide to channels, skills, automations, and real workflows.

[![GitHub stars](https://img.shields.io/github/stars/opencudem/openclaw-101?style=social)](https://github.com/opencudem/openclaw-101)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Discord](https://img.shields.io/badge/Discord-Join%20Chat-7289DA?logo=discord)](https://discord.com/invite/clawd)

[🚀 Quick Start](#get-started-in-5-minutes) · [📚 Learning Path](#learning-path) · [🛠️ Catalog](CATALOG.md) · [💡 Use Cases](USE_CASES.md)

---

## Why this guide exists

OpenClaw is powerful — it connects to WhatsApp, Telegram, Discord, Slack, and more. It can browse the web, run commands, manage your calendar, and automate recurring tasks. But that breadth can be overwhelming.

This guide is the **opinionated path** we wish existed when we started: from first install to useful workflows, in a predictable sequence.

## Why this guide is different

| Official Docs | This Guide |
|--------------|------------|
| Comprehensive reference | Practical, progressive learning |
| Feature-focused | Workflow-focused |
| Assumes you know what you want | Helps you discover what's possible |
| Perfect for lookup | Perfect for onboarding |

Use them together: start here, then use the official docs as reference.

## Get started in 5 minutes

```bash
# 1. Install OpenClaw Gateway
npm install -g openclaw

# 2. Start the Gateway
openclaw gateway start

# 3. Connect your first channel
openclaw channel add telegram

# 4. Chat with your agent
# Send "hello" to your Telegram bot
```

👉 Continue to [01-getting-started](01-getting-started/) for full setup details.

## Learning Path

| Level | Modules | Time | Goal |
|-------|---------|------|------|
| 🌱 Beginner | 01-03 | 2-3 hours | Get OpenClaw running with your favorite chat app |
| 🌿 Intermediate | 04-06 | 4-6 hours | Teach it skills and automate daily tasks |
| 🌳 Advanced | 07-11 | 6-8 hours | Build complex workflows and multi-agent systems |
| 🔧 Maintenance | 12 | Ongoing | Proactive monitoring and documentation updates |

**Weekend goal:** Complete Beginner + Intermediate for immediate daily value.

## Module Map

| Module | What you'll learn | Time |
|--------|------------------|------|
| [01-getting-started](01-getting-started/) | Installation, first channel, basic commands | 30 min |
| [02-channels](02-channels/) | WhatsApp, Telegram, Discord, Slack setup | 45 min |
| [03-memory](03-memory/) | Personalization, context, file-based memory | 30 min |
| [04-skills](04-skills/) | Skill architecture, ClawHub, local vs workspace | 60 min |
| [05-integrations](05-integrations/) | GitHub, calendar, email, APIs | 45 min |
| [06-automation](06-automation/) | Cron jobs, recurring tasks, briefings | 45 min |
| [07-browser-automation](07-browser-automation/) | Research, form filling, data extraction | 60 min |
| [08-workflows](08-workflows/) | Combining everything into real systems | 90 min |
| [09-security-and-ops](09-security-and-ops/) | Approvals, permissions, reliability | 45 min |
| [10-cli-and-reference](10-cli-and-reference/) | Complete command reference | 30 min |
| [11-creating-agents](11-creating-agents/) | Multi-agent architecture and routing | 45 min |
| [12-heartbeat](12-heartbeat/) | Proactive automation and maintenance | 15 min |

## Flagship Workflows

Ready-to-deploy examples combining multiple features:

| Workflow | Components | Difficulty |
|----------|------------|------------|
| [Morning Briefing](examples/personal-productivity/morning-briefing.md) | Cron + Channels + Weather + Calendar | 🟢 Easy |
| [Engineering Inbox](examples/engineering/pr-alerts.md) | Discord/Slack + GitHub skill | 🟡 Medium |
| [Personal Follow-up System](examples/founder-ops/follow-up-system.md) | Memory + Messaging + Reminders | 🟡 Medium |
| [Research Agent](examples/research-and-content/research-agent.md) | Browser + Search + Summarization | 🔴 Advanced |
| [Approval-Gated Actions](09-security-and-ops/approval-patterns.md) | Channels + Approvals + Isolation | 🔴 Advanced |

👉 See all workflows in [examples/](examples/)

## Resource Docs

- [📖 CATALOG.md](CATALOG.md) — Curated skills and integrations
- [🗺️ LEARNING-ROADMAP.md](LEARNING-ROADMAP.md) — What's coming next
- [⚡ QUICK_REFERENCE.md](QUICK_REFERENCE.md) — Cheat sheet for daily use
- [🔧 TROUBLESHOOTING.md](TROUBLESHOOTING.md) — Common issues and fixes
- [💡 USE_CASES.md](USE_CASES.md) — Map your goal to the right module

## FAQ

**Q: Do I need coding experience?**
A: For basic usage, no. For custom skills and workflows, basic JavaScript/TypeScript helps.

**Q: Is this official OpenClaw documentation?**
A: No, this is a community guide. Official docs are at [docs.openclaw.ai](https://docs.openclaw.ai).

**Q: Can I contribute?**
A: Yes! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Contributing

We welcome contributions! Whether it's fixing a typo, adding a workflow, or creating a new module:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a PR

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

---

## Support / Star / Share

If this guide helped you:
- ⭐ Star this repo
- 🐛 [Open an issue](https://github.com/opencudem/openclaw-101/issues) for bugs or suggestions
- 💬 Join the [Discord community](https://discord.com/invite/clawd)
- 🐦 Share on social media

---

<p align="center">Made with 🐱 for the OpenClaw community</p>
