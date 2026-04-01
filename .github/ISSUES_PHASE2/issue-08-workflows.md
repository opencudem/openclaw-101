## [Phase 2] Fix 08-workflows: Update workflow examples with real commands

### Problem
Workflow examples likely use invented commands from other modules. Need to rewrite with verified syntax.

### Affected Workflows
1. Morning Briefing (personal-productivity)
2. PR Alerts (engineering)
3. Follow-up System (founder-ops)
4. Research Agent (research-and-content)

### Research Source
- Research document: `openclaw-101-research/RESEARCH.md`
- All corrected module documentation

### Tasks
- [ ] **Morning Briefing:**
  - Fix cron syntax to use --name, --cron, --message, --announce
  - Verify weather/calendar skills exist on ClawHub
  - Use real skill slugs
- [ ] **PR Alerts:**
  - Fix GitHub skill integration
  - Verify `openclaw skills install github` works
  - Correct cron job syntax
  - Fix notification delivery syntax
- [ ] **Follow-up System:**
  - Fix memory storage paths
  - Verify persistence mechanism
  - Correct any invented commands
- [ ] **Research Agent:**
  - Fix browser commands
  - Verify `openclaw browser` subcommands
  - Correct search/scrape workflow
- [ ] General:
  - Replace all invented `skills.*` calls with real patterns
  - Use actual OpenClaw mechanisms only
  - Test workflow logic against docs

### Acceptance Criteria
- [ ] All workflows use verified commands only
- [ ] No invented skill invocations
- [ ] Cron syntax correct in all examples
- [ ] Channel commands correct
- [ ] All examples are copy-paste ready with real commands

### Priority
🟡 **Medium** - Dependent on other fixes

### Estimated Effort
Medium (3-4 hours)

### Dependencies
- Issue #6 (automation/cron) must be fixed first
- Issue #2 (channels) should be fixed first
- Issue #4 (skills) should be fixed first

---
**Created:** 2026-04-01
**Depends on:** Other Phase 2 issues
