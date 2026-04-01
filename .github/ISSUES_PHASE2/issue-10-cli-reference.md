## [Phase 2] Fix 10-cli-and-reference: Correct all CLI commands

### Problem
CLI reference has many incorrect commands. This is the lookup module - it must be 100% accurate.

### Current Issues
- `openclaw channel *` → should be `openclaw channels *`
- `openclaw skill *` → should be `openclaw skills *`
- `openclaw gateway start` → should be `openclaw gateway`
- Port 3000 → should be 18789
- Missing commands: pairing, agents, plugins, browser
- Missing flags: --require-rpc, --announce, etc.

### Research Source
- Research document: `openclaw-101-research/RESEARCH.md` (lines 400-450)
- Official docs: https://docs.openclaw.ai/cli/index
- Official docs: https://docs.openclaw.ai/llms.txt

### Tasks
- [ ] Fix all `channel` → `channels` (plural)
- [ ] Fix all `skill` → `skills` (plural)
- [ ] Fix gateway commands section
- [ ] Fix port: 3000 → 18789 everywhere
- [ ] Add missing command sections:
  - `openclaw pairing list/approve`
  - `openclaw agents add/bind/set-identity`
  - `openclaw plugins list/install/enable`
  - `openclaw browser status/navigate/click`
  - `openclaw memory status/index/search`
  - `openclaw doctor/doctor --fix`
  - `openclaw security audit`
- [ ] Add global flags: --dev, --profile, --json, --no-color
- [ ] Update environment variables list
- [ ] Update file locations
- [ ] Cross-reference every command with official docs

### Acceptance Criteria
- [ ] 100% of commands verified against official docs
- [ ] Every command tested (where possible)
- [ ] No invented commands
- [ ] Complete command tree from `openclaw --help`
- [ ] Accurate flags for each command

### Priority
🔴 **Critical** - This is the reference document

### Estimated Effort
Medium (3-4 hours)

---
**Created:** 2026-04-01
**Depends on:** Research completed in `openclaw-101-research/RESEARCH.md`
