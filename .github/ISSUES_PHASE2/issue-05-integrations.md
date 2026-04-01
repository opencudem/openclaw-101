## [Phase 2] Fix 05-integrations: Remove invented integrations, document real ones

### Problem
Integration module may contain invented skills (github, calendar, etc.). Need to verify against actual ClawHub registry.

### Research Source
- ClawHub registry: https://clawhub.ai
- Official docs: https://docs.openclaw.ai/tools/clawhub
- Command: `openclaw skills search <query>`

### Tasks
- [ ] Audit all listed integrations against ClawHub:
  - Check if github skill exists
  - Check if calendar skill exists
  - Check if email skill exists
  - Check if postgres/sqlite skills exist
  - Check if notion/linear/trello skills exist
- [ ] Remove invented integrations that don't exist
- [ ] Document how to find real skills:
  ```bash
  openclaw skills search "github"
  openclaw skills search "calendar"
  openclaw skills search "database"
  ```
- [ ] Document configuration via openclaw.json
- [ ] Document SecretRef for secure credentials:
  ```json
  {
    "source": "env",
    "provider": "default",
    "id": "API_KEY_NAME"
  }
  ```
- [ ] Add examples of real skill configurations
- [ ] Link to ClawHub for discovery

### Acceptance Criteria
- [ ] All listed integrations verified on ClawHub
- [ ] Invented integrations removed
- [ ] Real discovery process documented
- [ ] SecretRef for credentials explained
- [ ] No fake skill slugs remain

### Priority
🟡 **Medium** - Important but not core functionality

### Estimated Effort
Medium (2-3 hours) - Requires ClawHub verification

---
**Created:** 2026-04-01
**Depends on:** Research and ClawHub access
