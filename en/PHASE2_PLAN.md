# Phase 2: Content Correction Plan

**Status:** Planned  
**Priority:** High  
**Goal:** Fix all invented/incorrect commands in openclaw-101 based on verified research

---

## Overview

Based on comprehensive research of official OpenClaw documentation, the initial content contains several invented commands and incorrect information. This plan outlines all corrections needed, organized as trackable GitHub issues.

## Critical Corrections Summary

| Incorrect (Current) | Correct (Official) |
|---------------------|----------------------|
| `openclaw init` | `openclaw onboard --install-daemon` |
| `openclaw gateway start` | `openclaw gateway` or `openclaw gateway run` |
| Port 3000 | **Port 18789** |
| `openclaw channel add <name>` | `openclaw channels add --channel <name> --token <token>` |
| `openclaw skill install` | `openclaw skills install <slug>` |
| `openclaw cron add "name"` | `openclaw cron add --name "..." --cron "..."` |

---

## GitHub Issues to Create

### Issue #1: Fix 01-getting-started Module
**Title:** `[Phase 2] Fix 01-getting-started: Correct onboarding and gateway commands`

**Description:**
The getting started module contains incorrect commands that don't exist in OpenClaw.

**Tasks:**
- [ ] Replace `openclaw init` with `openclaw onboard --install-daemon`
- [ ] Add onboarding flags documentation (--flow, --mode, --non-interactive)
- [ ] Fix gateway commands: `openclaw gateway` not `openclaw gateway start`
- [ ] Correct port: 18789 not 3000
- [ ] Verify dashboard command is correct (`openclaw dashboard`)
- [ ] Add verification steps with `openclaw gateway status`
- [ ] Test commands against actual OpenClaw installation

**Priority:** 🔴 Critical

---

### Issue #2: Fix 02-channels Module
**Title:** `[Phase 2] Fix 02-channels: Correct channel command syntax`

**Description:**
Channel commands use wrong syntax. Should be `channels` (plural) with --channel flag.

**Tasks:**
- [ ] Fix: `openclaw channel add` → `openclaw channels add --channel <name> --token <token>`
- [ ] Update Telegram section with bot token setup
- [ ] Update Discord section with intents and privileged gateway setup
- [ ] Add WhatsApp pairing flow documentation
- [ ] Add Slack Socket Mode vs HTTP mode explanation
- [ ] Document DM policies: pairing, allowlist, open, disabled
- [ ] Add pairing commands: `openclaw pairing list/approve <channel> <code>`
- [ ] Fix all example commands to match official syntax

**Priority:** 🔴 Critical

---

### Issue #3: Fix 03-memory Module
**Title:** `[Phase 2] Verify 03-memory: Ensure content matches official docs`

**Description:**
Verify memory module aligns with official OpenClaw memory documentation.

**Tasks:**
- [ ] Verify memory locations: MEMORY.md, memory/*.md, ~/.openclaw/memory/
- [ ] Check if `openclaw memory` commands are correct (status, index, search)
- [ ] Verify memory search behavior
- [ ] Cross-reference with https://docs.openclaw.ai/concepts/memory
- [ ] Fix any invented concepts
- [ ] Add actual CLI examples for memory operations

**Priority:** 🟡 Medium

---

### Issue #4: Fix 04-skills Module
**Title:** `[Phase 2] Fix 04-skills: Correct skill installation and AgentSkills format`

**Description:**
Skills module has incorrect commands and incomplete AgentSkills format documentation.

**Tasks:**
- [ ] Fix: `openclaw skill install` → `openclaw skills install <slug>`
- [ ] Fix: `openclaw skill list` → `openclaw skills list`
- [ ] Document skill precedence order correctly
- [ ] Add SKILL.md format with metadata.openclaw.requires
- [ ] Document gating: requires.bins, requires.env, requires.config
- [ ] Add `openclaw skills search/info/check` commands
- [ ] Document skill locations and precedence
- [ ] Add `clawhub sync` for publishing workflows
- [ ] Cross-reference with https://docs.openclaw.ai/tools/skills

**Priority:** 🔴 Critical

---

### Issue #5: Fix 05-integrations Module
**Title:** `[Phase 2] Fix 05-integrations: Remove invented integrations, document real ones`

**Description:**
Integration module may contain invented skills. Need to verify against ClawHub.

**Tasks:**
- [ ] Verify which skills actually exist on ClawHub (clawhub.ai)
- [ ] Remove invented integrations (github, calendar, etc. if not real)
- [ ] Document how to find real skills: `openclaw skills search <query>`
- [ ] Add configuration via openclaw.json or env vars
- [ ] Document SecretRef for secure credential storage
- [ ] Cross-reference with actual ClawHub registry

**Priority:** 🟡 Medium

---

### Issue #6: Fix 06-automation Module
**Title:** `[Phase 2] Fix 06-automation: Correct cron job syntax and concepts`

**Description:**
Cron module has completely wrong command syntax and invented concepts.

**Tasks:**
- [ ] Fix cron add syntax: `openclaw cron add --name "..." --cron "..." --message "..."`
- [ ] Document `--session` flag: main vs isolated vs custom
- [ ] Document `--announce`, `--channel`, `--to` for delivery
- [ ] Add `--at` for one-shot jobs
- [ ] Fix `openclaw cron runs --id <job-id>` to check status
- [ ] Document job storage: ~/.openclaw/cron/jobs.json
- [ ] Add retry policy: 30s → 1m → 5m → 15m → 60m
- [ ] Remove invented concepts (session types as commands)
- [ ] Cross-reference with https://docs.openclaw.ai/automation/cron-jobs

**Priority:** 🔴 Critical

---

### Issue #7: Fix 07-browser-automation Module
**Title:** `[Phase 2] Verify 07-browser-automation: Ensure commands match openclaw browser`

**Description:**
Verify browser automation uses correct `openclaw browser` commands.

**Tasks:**
- [ ] Verify `openclaw browser` subcommands (status, start, stop, navigate, click, etc.)
- [ ] Check if screenshot, type, fill, press commands are correct
- [ ] Document tabs management: `openclaw browser tabs/open/close`
- [ ] Cross-reference with https://docs.openclaw.ai/cli/browser
- [ ] Fix any invented patterns

**Priority:** 🟡 Medium

---

### Issue #8: Fix 08-workflows Module
**Title:** `[Phase 2] Fix 08-workflows: Update workflow examples with real commands`

**Description:**
Workflow examples likely use invented commands. Need to rewrite with verified syntax.

**Tasks:**
- [ ] Rewrite Morning Briefing with real cron syntax
- [ ] Rewrite PR Alerts with real skills/channels commands
- [ ] Rewrite Follow-up System with real memory commands
- [ ] Rewrite Research Agent with real browser commands
- [ ] Ensure all code examples use verified commands only
- [ ] Test workflow logic against official docs

**Priority:** 🟡 Medium

---

### Issue #9: Fix 09-security-and-ops Module
**Title:** `[Phase 2] Fix 09-security-and-ops: Remove invented approval patterns`

**Description:**
Security module contains invented approval system that doesn't exist in OpenClaw.

**Tasks:**
- [ ] Remove invented `skills.approval.request` pattern (doesn't exist)
- [ ] Document real security features: `openclaw security audit`
- [ ] Add DM/group policies as actual access control
- [ ] Document pairing system as security mechanism
- [ ] Add `openclaw doctor --fix` for security repairs
- [ ] Cross-reference with https://docs.openclaw.ai/gateway/security
- [ ] Rewrite approval patterns to use real OpenClaw mechanisms

**Priority:** 🔴 Critical

---

### Issue #10: Fix 10-cli-and-reference Module
**Title:** `[Phase 2] Fix 10-cli-and-reference: Correct all CLI commands`

**Description:**
CLI reference has many incorrect commands that need fixing.

**Tasks:**
- [ ] Fix all channel commands to use `channels` (plural)
- [ ] Fix all skill commands to use `skills` (plural)
- [ ] Fix cron commands with correct flag syntax
- [ ] Add missing commands: pairing, agents, plugins, browser
- [ ] Correct environment variables list
- [ ] Fix file locations (port 18789, correct paths)
- [ ] Cross-reference with https://docs.openclaw.ai/cli/index
- [ ] Ensure every command is verified

**Priority:** 🔴 Critical

---

### Issue #11: Update Resource Documentation
**Title:** `[Phase 2] Update resource docs: CATALOG, QUICK_REFERENCE, TROUBLESHOOTING`

**Description:**
Resource documents need updates to match corrected commands.

**Tasks:**
- [ ] Update CATALOG.md with real ClawHub skills only
- [ ] Update QUICK_REFERENCE.md with correct commands
- [ ] Update TROUBLESHOOTING.md with real diagnostic commands
- [ ] Add `openclaw doctor` troubleshooting flow
- [ ] Update USE_CASES.md to map to real modules
- [ ] Cross-reference all commands with research document

**Priority:** 🟡 Medium

---

### Issue #12: Create Verification Checklist
**Title:** `[Phase 2] Create command verification checklist for future content`

**Description:**
Create a process to prevent invented commands in future content.

**Tasks:**
- [ ] Create CHECKLIST.md for content verification
- [ ] Document rule: Always verify commands against docs.openclaw.ai
- [ ] Add template for SKILL.md verification
- [ ] Create script to extract and validate commands
- [ ] Document research workflow in CONTRIBUTING.md
- [ ] Add CI check for command validation (if possible)

**Priority:** 🟢 Low

---

## Implementation Order

### Phase 2A: Critical Fixes (Week 1)
1. Issue #1: Fix 01-getting-started
2. Issue #2: Fix 02-channels
3. Issue #4: Fix 04-skills
4. Issue #6: Fix 06-automation
5. Issue #9: Fix 09-security-and-ops
6. Issue #10: Fix 10-cli-and-reference

### Phase 2B: Medium Priority (Week 2)
7. Issue #3: Verify 03-memory
8. Issue #5: Fix 05-integrations
9. Issue #7: Verify 07-browser-automation
10. Issue #8: Fix 08-workflows

### Phase 2C: Polish (Week 3)
11. Issue #11: Update resource docs
12. Issue #12: Create verification checklist

---

## Definition of Done

For each issue:
- [ ] All invented commands removed
- [ ] All commands verified against https://docs.openclaw.ai
- [ ] Research document referenced for each command
- [ ] Code examples tested (where possible)
- [ ] PR submitted with clear description of changes
- [ ] Cross-references added to official docs

---

## Research Source

All fixes should reference:
- `/home/hoang_kieng_io_vn/.openclaw/workspace/openclaw-101-research/RESEARCH.md`
- Official docs: https://docs.openclaw.ai

---

**Created:** 2026-04-01  
**Research completed by:** Mina 🐱
