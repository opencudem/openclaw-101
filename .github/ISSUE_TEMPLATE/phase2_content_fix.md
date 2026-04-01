# GitHub Issue Template: Content Correction

Use this template to create GitHub issues for Phase 2 content corrections.

---

## Issue Template

```markdown
## [Phase 2] Fix [MODULE_NAME]: [Brief Description]

### Problem
The [module name] module contains incorrect/invented commands that don't match official OpenClaw documentation.

### Specific Issues
- [ ] Command A is incorrect: `wrong command` → should be `correct command`
- [ ] Command B is invented and doesn't exist
- [ ] Concept C is not part of OpenClaw

### Research Source
- Research document: `/openclaw-101-research/RESEARCH.md`
- Official docs: https://docs.openclaw.ai/[relevant-path]

### Tasks
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

### Acceptance Criteria
- [ ] All commands verified against official docs
- [ ] No invented concepts remain
- [ ] Examples use real commands only
- [ ] Cross-references added to official docs

### Priority
🔴 Critical / 🟡 Medium / 🟢 Low

### Estimated Effort
Small / Medium / Large
```

---

## Quick Links for Issue Creation

Create issues at: https://github.com/opencudem/openclaw-101/issues/new

### Pre-filled Issue Titles:

1. `[Phase 2] Fix 01-getting-started: Correct onboarding and gateway commands`
2. `[Phase 2] Fix 02-channels: Correct channel command syntax`
3. `[Phase 2] Verify 03-memory: Ensure content matches official docs`
4. `[Phase 2] Fix 04-skills: Correct skill installation and AgentSkills format`
5. `[Phase 2] Fix 05-integrations: Remove invented integrations`
6. `[Phase 2] Fix 06-automation: Correct cron job syntax and concepts`
7. `[Phase 2] Verify 07-browser-automation: Ensure commands match openclaw browser`
8. `[Phase 2] Fix 08-workflows: Update workflow examples with real commands`
9. `[Phase 2] Fix 09-security-and-ops: Remove invented approval patterns`
10. `[Phase 2] Fix 10-cli-and-reference: Correct all CLI commands`
11. `[Phase 2] Update resource docs: CATALOG, QUICK_REFERENCE, TROUBLESHOOTING`
12. `[Phase 2] Create command verification checklist for future content`

---

## Labels to Use

Suggested labels for these issues:
- `phase-2`
- `content-fix`
- `documentation`
- `bug` (for critical issues)
- `enhancement` (for verification checklist)

---

## Milestone

Create milestone: **Phase 2: Content Corrections**
- Start: 2026-04-01
- Target: 2026-04-21 (3 weeks)

---

## Assignment

Recommended assignment by priority:

**Critical (do first):**
- Issues 1, 2, 4, 6, 9, 10

**Medium (do next):**
- Issues 3, 5, 7, 8, 11

**Low (do last):**
- Issue 12
