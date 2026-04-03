# DocLoop 6-Phase Workflow

> Complete reference for automated OpenClaw 101 documentation maintenance.
> This file is included in HEARTBEAT.md for detailed lookup during workflow execution.

---

## Overview

DocLoop is a systematic 6-phase workflow designed for maintaining the `opencudem/openclaw-101` documentation repository. It ensures documentation stays accurate, complete, and aligned with the latest OpenClaw releases.

**Triggers:**
1. **Periodic heartbeat** — scheduled automated runs
2. **Manual request** — human-triggered maintenance
3. **Post-release** — triggered after new OpenClaw releases

---

## Phase 1 — SCAN (Discovery)

### Purpose
Identify new information sources and gather raw research data before any documentation decisions are made.

### Sources to Check
| Source | What to Look For | Tool |
|--------|------------------|------|
| GitHub `openclaw/openclaw` releases | New versions, changelogs, breaking changes | web_fetch |
| `docs.openclaw.ai` | Documentation updates, new guides | firecrawl scrape |
| `awesome-openclaw-skills` | New community skills | web_fetch + exa search |
| `awesome-openclaw-agents` | New agent patterns | web_fetch + exa search |
| Reddit r/openclaw | Popular use cases, common questions | exa search |
| Discord community | Feature requests, pain points | exa search |

### Output Structure
All content organized in `/home/hoang_kieng_io_vn/.openclaw/workspace/openclaw-101-research/`:

```
openclaw-101-research/
├── README.md              # Master research index
├── releases/              # GitHub release notes by version
├── docs/                  # Scraped docs.openclaw.ai pages
├── awesome-skills.md      # Catalog of new skills
├── awesome-agents.md      # Catalog of new agents
├── community/             # Reddit, Discord findings
│   ├── reddit.md
│   └── discord.md
└── .qmd/                  # QMD index for searchability
```

### Post-Scan Action
- Run QMD indexing on all collected content
- Verify research directory is searchable

---

## Phase 2 — AUDIT (Gap Analysis)

### Purpose
Compare scan findings against existing documentation to identify gaps and outdated content.

### Audit Dimensions

| Dimension | Check | Scoring |
|-----------|-------|---------|
| **Freshness** | Last update vs. release date | 1-5 (5 = current) |
| **Accuracy** | Config syntax matches current version | 1-5 (5 = verified correct) |
| **Level Coverage** | Beginner/Intermediate/Advanced coverage | B/I/A tags |
| **Examples** | Presence of working, runnable code | Yes/No + completeness |
| **Visuals** | Screenshots for dashboard/setup flows | Yes/No + `[SCREENSHOT NEEDED]` flags |

### Audit Checklist per Module
- [ ] New features documented?
- [ ] Deprecated APIs flagged?
- [ ] Old file paths updated?
- [ ] Version references current?
- [ ] CLI commands verified against `openclaw --help`?

### Output
**`AUDIT_REPORT.md`** with:
- Scored priority list (1-5 scale)
- Categorized work items: New | Update | Deprecate | Visual
- Estimated effort per item

---

## Phase 3 — RESEARCH (Evidence Gathering)

### Purpose
Gather authoritative evidence for each work item before writing. **Official docs are ground truth.**

### Per-Item Research Process

#### Step 1: Fetch Authoritative Sources
- Official docs at `docs.openclaw.ai/{topic}`
- GitHub source code for implementation details
- Skill `SKILL.md` files for tool behavior

#### Step 2: Validate Syntax
- Check config keys against current `openclaw.json` schema
- Verify CLI flags with `openclaw <command> --help`
- Test examples in isolated environment when possible

#### Step 3: Collect Community Examples
- Search Exa for "OpenClaw {feature} example"
- Check GitHub Gists for working configurations
- Review Reddit threads for real-world patterns

#### Step 4: Tag User Level
| Level | Criteria |
|-------|----------|
| **Beginner** | First-time setup, core concepts, no prerequisites |
| **Intermediate** | Requires working setup, multiple steps, some context |
| **Advanced** | Complex multi-agent, custom skills, edge cases |

### Research Tools
- `web_fetch` for official docs
- `exa` for community search
- `qmd` for searching indexed research
- File search for deep inspection of existing research files

### Output
Research notes per work item, linked to sources, with confidence ratings.

---

## Phase 4 — CREATE/UPDATE (Content Production)

### Purpose
Produce or update documentation following strict quality rules.

### For New Features

#### Structure
```markdown
# XX - Feature Name

> One-line summary of what this solves.

## What this module solves
Brief explanation of the problem/feature.

## When to use this module
Bullet list of scenarios.

## Prerequisites
- OpenClaw Version: vX.XX+
- Required: [dependencies]

## Quick Start
### Step 1: [Action]
```bash
# Complete, runnable command
openclaw example command --flag value
```

## Common Mistakes
| Mistake | Why It's Bad | Solution |
|---------|--------------|----------|
| Wrong thing | Explanation | Fix |

## CLI Reference
```bash
openclaw command --help
```

## Related Modules
- Previous: [XX-xxx](../XX-xxx/)
- Next: [XX-xxx](../XX-xxx/)
```

#### Content Quality Rules (Enforced)
- ✅ Every config example must be **complete and runnable** (no `...` gaps)
- ✅ Every guide must have a **"What you need"** prerequisites section
- ✅ All skill references must **link to ClawHub** or `awesome-openclaw-skills`
- ✅ Language: **clear, plain English**; no unexplained jargon
- ✅ Flag images: `[SCREENSHOT NEEDED: description]`
- ✅ Note version introduced: `(since vX.XX)`

### For Outdated Content
- Update version references
- Replace deprecated config keys
- Update changed file paths
- Add compatibility notes: "For OpenClaw < vX.XX, use..."
- Validate all examples against current CLI syntax

### Output
Draft documentation files ready for validation.

---

## Phase 5 — VALIDATE (Keep-or-Discard Gate)

### Purpose
Quality gate before commit. This is the **Karpathy BPB-style check** — if it doesn't meet standards, it doesn't ship.

### Validation Checklist

| Check | Method | Pass Criteria |
|-------|--------|---------------|
| Config keys | Cross-reference against official docs | All keys verified |
| URLs live | `web_fetch` or `curl` | 200 OK |
| Version numbers | Match Phase 1 scan report | Exact match |
| No secrets | Regex scan for tokens, keys | Zero findings |
| No duplication | QMD search against existing docs | No matches |
| Examples complete | Visual scan for `...` gaps | None found |
| Prerequisites present | Check for "What you need" section | Exists |
| Links valid | Click test or fetch | All resolve |

### Keep-or-Discard Logic
```
IF all checks pass:
    → Include in commit
ELSE:
    → Move to PENDING_REVIEW.md
    → Document failure reason
    → Exclude from commit
    → Human decides: fix now | defer | discard
```

### Output
- Validated files staged for commit
- `PENDING_REVIEW.md` with flagged items and reasons

---

## Phase 6 — COMMIT & REPORT (Delivery)

### Purpose
Ship validated changes and notify stakeholders.

### Step 1: Create Branch
```bash
git checkout -b claude/docloop-YYYYMMDD-HHmm
```

### Step 2: Commit with Structured Message
```
docs(docloop): update for openclaw v2026.X.X

- Added:
  - 11-creating-agents: Discord multi-agent auth section
  - XX-new-feature: Complete guide
  
- Updated:
  - 02-channels: Fixed CLI commands
  - 05-integrations: Updated version references
  
- Flagged (see PENDING_REVIEW.md):
  - 09-security: Needs screenshot verification
  - 10-cli: Advanced section needs expert review

Scan source: openclaw/openclaw@v2026.X.X
Loop run: 2026-04-03T12:00:00Z
```

### Step 3: Open Pull Request
- Target: `main`
- Include summary table:
  | Type | Count | Files |
  |------|-------|-------|
  | New guides | 3 | list |
  | Updated | 7 | list |
  | Needs review | 2 | list |

- Labels: `automated`, `documentation`, `docloop`

### Step 4: Send Report
**To Telegram/Discord:**
```
📚 DocLoop Run Complete — openclaw v2026.X.X

✅ New guides: 3
🔄 Updated: 7 files
⚠️ Needs review: 2 items (see PENDING_REVIEW.md)

🔗 PR: github.com/opencudem/openclaw-101/pull/XX
📋 Full details: openclaw-101-research/AUDIT_REPORT.md
```

---

## Rules (Non-Negotiable)

| # | Rule | Violation Consequence |
|---|------|----------------------|
| 1 | **Official docs = ground truth only** — NO invention | Content rejected, flagged for human review |
| 2 | **Never push to main** — always PR | Rollback required if violated |
| 3 | **Keep-or-discard gate** — failed validation → PENDING_REVIEW.md | Unvalidated content never ships |
| 4 | **All CLI commands must be verified** against live docs | Command errors flagged as accuracy failure |

---

## Tool Inventory

| Tool | Purpose | When to Use |
|------|---------|-------------|
| `web_fetch` | Fetch official docs pages | Phase 1, 3, 5 |
| `firecrawl` | Deep scrape entire sites | Phase 1 (docs, awesome repos) |
| `exa` | Community search | Phase 1, 3 |
| `qmd` | Index & search research | Phase 1 (end), 3, 5 |
| `web_search` | Quick fact verification | All phases |
| Git | Branch, commit, PR | Phase 6 |

---

## Artifact Reference

| Artifact | Phase Created | Purpose |
|----------|---------------|---------|
| `openclaw-101-research/` | 1 | Raw research data |
| `AUDIT_REPORT.md` | 2 | Prioritized work list |
| Research notes (per item) | 3 | Evidence for writing |
| Draft docs | 4 | Content ready for validation |
| `PENDING_REVIEW.md` | 5 | Flagged items for human |
| Git branch | 6 | Staged changes |
| Discord/Telegram report | 6 | Stakeholder notification |

---

## Quick Reference: Phase Entry Conditions

| Phase | Entry Condition | Exit Condition |
|-------|-----------------|----------------|
| 1 | Trigger fired | Research directory populated & indexed |
| 2 | Phase 1 complete | AUDIT_REPORT.md generated |
| 3 | Phase 2 complete | All work items researched |
| 4 | Phase 3 complete | Draft content written |
| 5 | Phase 4 complete | Content validated or flagged |
| 6 | Phase 5 complete | PR opened, report sent |

---

_Last updated: 2026-04-03_
_Workflow version: 1.0_
