# Contributing to OpenClaw-101

Thank you for your interest in contributing! This guide helps you get started.

## Ways to Contribute

- **Fix typos** - Found an error? PRs welcome!
- **Add modules** - Deep dives on specific topics
- **Create workflows** - Real-world examples
- **Improve examples** - Better code, clearer explanations
- **Translate** - Make it accessible in more languages ([see TRANSLATION.md](TRANSLATION.md))

## Getting Started

1. Fork the repository
2. Clone your fork
3. Create a branch: `git checkout -b my-feature`
4. Make changes
5. Commit: `git commit -am "Add feature"`
6. Push: `git push origin my-feature`
7. Open a Pull Request

## Content Guidelines

### Writing Style

- **Clear over clever** - Simple explanations win
- **Show, don't tell** - Include working code
- **Progressive** - Beginner → Intermediate → Advanced
- **Practical** - Focus on real use cases

### Module Structure

Each module should follow this template:

```markdown
# Module Title

## What this module solves
Brief description of the problem.

## When to use this module
Specific use cases.

## Quick Start
Get running in 5 minutes.

## How it works under the hood
Architecture and concepts.

## Copy-paste examples
Working code snippets.

## Common mistakes and troubleshooting
Help users avoid pitfalls.

## Advanced patterns
Deeper techniques.

## Related modules and next step
Where to go next.
```

### Code Examples

- Include complete, runnable examples
- Add comments for clarity
- Test before submitting
- Use real-world scenarios

## Workflow Contributions

New workflows go in `examples/<category>/`:

```
examples/
├── founder-ops/
├── engineering/
├── personal-productivity/
└── research-and-content/
```

Each workflow needs:

1. **README.md** - Full documentation
2. **Workflow script** - The code
3. **Configuration** - Any required setup
4. **Screenshots** - If applicable (save to `resources/screenshots/`)

### Workflow Template

```markdown
# Workflow Name

## What it does
One-line description.

## Components used
- Channel: Telegram
- Skills: weather, calendar
- Automation: Cron

## Setup
Step-by-step instructions.

## Usage
How to trigger it.

## Customization
How to adapt for your needs.
```

## Review Process

1. PR submitted
2. Automated checks run
3. Maintainer review
4. Feedback or merge

## Questions?

- Open an issue for discussion
- Join [Discord](https://discord.com/invite/clawd)
- Check existing PRs for examples

## Code of Conduct

- Be respectful
- Help others learn
- Focus on constructive feedback
- Assume good intent

## License

By contributing, you agree your work will be licensed under MIT License.

---

Thank you for helping make OpenClaw-101 better! 🐱
