# Translation Guide for OpenClaw 101

Thank you for helping make OpenClaw accessible worldwide! This guide covers how to add or improve translations.

## Quick Start

1. Fork the repository
2. Create a language folder (e.g., `vi/` for Vietnamese)
3. Copy the structure from `en/`
4. Translate content
5. Submit a PR

## Folder Structure

Each language has its own complete copy:

```
openclaw-101/
├── README.md              # Language selector (root)
├── en/                    # English (source)
│   ├── README.md
│   ├── 01-getting-started/
│   ├── 02-channels/
│   └── ...
├── vi/                    # Vietnamese
│   ├── README.md
│   ├── 01-getting-started/
│   └── ...
└── TRANSLATION.md         # This file
```

## What to Translate

### Required (Priority Order)

| Priority | File | Why |
|----------|------|-----|
| 1 | `README.md` | Entry point for all users |
| 2 | `01-getting-started/` | First impression, installation |
| 3 | `QUICK_REFERENCE.md` | Daily use reference |
| 4 | `02-channels/` | Core setup |
| 5 | `TROUBLESHOOTING.md` | Reduces support burden |

### Optional (Nice to Have)

- Individual module guides (03-12)
- Example workflows
- CATALOG.md, USE_CASES.md
- CONTRIBUTING.md, CHANGELOG.md

## Translation Principles

### Keep Structure Identical

- Same folder names (`01-getting-started`, not `01-bắt-đầu`)
- Same file names (`README.md`, not `GIỚI-THIỆU.md`)
- Same heading hierarchy
- Same code blocks (don't translate commands)

### What NOT to Translate

```markdown
# DON'T translate:
- Code blocks
- Command-line examples: `openclaw gateway start`
- File paths: `~/.openclaw/config`
- Environment variables: `OPENCLAW_PORT`
- Skill slugs: `weather`, `github`
- URLs (unless language-specific version exists)
- GitHub usernames, Discord channels

# DO translate:
- Explanations
- Comments in code
- UI labels mentioned in text
- Error messages (if explaining them)
- Navigation links between modules
```

### Technical Terms

Keep English terms that users will see in the actual software:

| English | Vietnamese Example | Notes |
|---------|-------------------|-------|
| Gateway | Gateway | Don't translate — it's the product name |
| Channel | Channel / Kênh | Either works, be consistent |
| Skill | Skill / Kỹ năng | Prefer "Skill" with explanation |
| Cron job | Cron job | Technical term, keep it |
| Bot token | Bot token | Users copy this from apps |
| Agent | Agent / Trợ lý | Pick one, stick with it |

## Translation Status Badge

Add this to your language README:

```markdown
**Translation Status:** 🚧 In Progress (3/12 modules)

**Contributors:** @your-github-username
**Last Updated:** 2026-04-03
```

## Review Checklist

Before submitting PR:

- [ ] All links work (check relative paths like `../02-channels/`)
- [ ] Code examples unchanged (only comments translated)
- [ ] Screenshots match if included
- [ ] No broken formatting (tables, code blocks)
- [ ] Spell check passed
- [ ] One other native speaker reviewed

## Example: Translating a Module

### Original (en/01-getting-started/README.md)

```markdown
## What this module solves

This module takes you from zero to a working OpenClaw Gateway 
with your first connected channel.

## Quick Start

```bash
# Install OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash
```

### Translated (vi/01-getting-started/README.md)

```markdown
## Module này giải quyết gì

Module này đưa bạn từ con số 0 đến một OpenClaw Gateway 
hoạt động với kênh đầu tiên được kết nối.

## Bắt đầu nhanh

```bash
# Cài đặt OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash
```
```

## Language-Specific Notes

### Vietnamese (vi)

- Use formal "bạn" instead of "mày/cậu"
- Keep technical terms in English with Vietnamese explanation
- Date format: DD/MM/YYYY
- Example phone numbers: Use +84 format

## Questions?

- Open an issue with label `translation`
- Join [Discord](https://discord.com/invite/clawd)
- Check existing translations for patterns

---

Thank you for making OpenClaw accessible to more people! 🌍
