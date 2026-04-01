## [Phase 2] Verify 07-browser-automation: Ensure commands match openclaw browser

### Problem
Verify browser automation uses correct `openclaw browser` subcommands.

### Research Source
- Official docs: https://docs.openclaw.ai/cli/browser
- Research document: `openclaw-101-research/RESEARCH.md` (lines 385-400)

### Tasks
- [ ] Verify `openclaw browser` subcommands:
  - `openclaw browser status`
  - `openclaw browser start`
  - `openclaw browser stop`
  - `openclaw browser tabs`
  - `openclaw browser open <url>`
  - `openclaw browser close <tab-id>`
  - `openclaw browser navigate <url>`
  - `openclaw browser screenshot`
  - `openclaw browser snapshot`
  - `openclaw browser click <selector>`
  - `openclaw browser type <selector> <text>`
  - `openclaw browser fill <selector> <value>`
  - `openclaw browser press <key>`
  - `openclaw browser wait <ms>`
  - `openclaw browser evaluate <script>`
- [ ] Check if all commands in the guide exist
- [ ] Fix any invented browser commands
- [ ] Document tabs management
- [ ] Add examples of real browser workflows
- [ ] Cross-reference with official docs

### Acceptance Criteria
- [ ] All browser commands verified
- [ ] No invented subcommands
- [ ] Examples use real syntax
- [ ] Tabs management documented
- [ ] Cross-references added

### Priority
🟡 **Medium** - Specialized feature

### Estimated Effort
Small (1-2 hours)

---
**Created:** 2026-04-01
**Depends on:** Research completed
