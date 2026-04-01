## [Phase 2] Update resource docs: CATALOG, QUICK_REFERENCE, TROUBLESHOOTING

### Problem
Resource documents need updates to match corrected commands from other modules.

### Affected Files
- `CATALOG.md` - May list invented skills
- `QUICK_REFERENCE.md` - Has incorrect commands
- `TROUBLESHOOTING.md` - May have invented diagnostic commands
- `USE_CASES.md` - May reference incorrect modules

### Tasks
- [ ] **CATALOG.md:**
  - Remove invented skills
  - Verify all listed skills exist on ClawHub
  - Add discovery section: `openclaw skills search`
  - Fix install commands
- [ ] **QUICK_REFERENCE.md:**
  - Fix all `channel` → `channels`
  - Fix all `skill` → `skills`
  - Fix port 3000 → 18789
  - Fix cron syntax
  - Add missing commands
  - Verify file locations
- [ ] **TROUBLESHOOTING.md:**
  - Fix diagnostic commands
  - Add `openclaw doctor` workflow
  - Add `openclaw logs --follow`
  - Add `openclaw gateway status --require-rpc`
  - Remove invented troubleshooting steps
- [ ] **USE_CASES.md:**
  - Update to reference correct modules
  - Fix any command examples
  - Verify goal-to-module mappings

### Acceptance Criteria
- [ ] All resource docs use correct commands
- [ ] No invented skills in CATALOG
- [ ] TROUBLESHOOTING uses real diagnostic commands
- [ ] QUICK_REFERENCE is actually quick and accurate

### Priority
🟡 **Medium** - Important reference materials

### Estimated Effort
Medium (3-4 hours)

### Dependencies
- Other Phase 2 issues should be completed first
- This is a final verification pass

---
**Created:** 2026-04-01
**Depends on:** Other Phase 2 issues
