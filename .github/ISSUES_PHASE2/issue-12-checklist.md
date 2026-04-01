## [Phase 2] Create command verification checklist for future content

### Problem
Need a process to prevent invented commands in future content additions.

### Goal
Create a system to ensure all future content uses verified commands only.

### Tasks
- [ ] Create `CHECKLIST.md` in root:
  ```markdown
  ## Content Verification Checklist
  
  Before submitting content:
  - [ ] All commands verified against docs.openclaw.ai
  - [ ] No commands invented or assumed
  - [ ] All skill slugs verified on clawhub.ai
  - [ ] All ports/defaults checked against official docs
  - [ ] Cross-references added to official documentation
  ```
- [ ] Update `CONTRIBUTING.md`:
  - Add verification requirements
  - Document research workflow
  - Add command validation section
- [ ] Create command extraction script:
  ```bash
  # Extract all commands from markdown
  grep -oP 'openclaw [a-z-]+' *.md | sort | uniq
  ```
- [ ] Create validation script idea:
  - Parse all `openclaw` commands from content
  - Compare against known valid commands
  - Flag unknown commands for review
- [ ] Document research workflow:
  1. Check https://docs.openclaw.ai/llms.txt
  2. Use `web_fetch` or `firecrawl` to get official docs
  3. Store research in `research/` folder
  4. Reference research in PR description
- [ ] Add to CI (if possible):
  - Lint for common invented patterns
  - Flag `openclaw init`, `openclaw channel add`, etc.

### Acceptance Criteria
- [ ] CHECKLIST.md created and documented
- [ ] CONTRIBUTING.md updated
- [ ] Research workflow documented
- [ ] Command extraction method provided
- [ ] Future content can be verified

### Priority
🟢 **Low** - Process improvement, not blocking

### Estimated Effort
Small (1-2 hours)

### Note
This prevents Phase 2 from happening again!

---
**Created:** 2026-04-01
**Can be done in parallel** with other Phase 2 work
