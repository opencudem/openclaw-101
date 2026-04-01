## [Phase 2] Fix 06-automation: Correct cron job syntax and concepts

### Problem
Cron module has completely wrong command syntax and invented concepts. Cron uses `--name`, `--cron`, `--message` flags, not positional arguments.

### Current Issues
| Current (Wrong) | Correct (Official) |
|-------------------|-------------------|
| `openclaw cron add "name"` | `openclaw cron add --name "..." --cron "..." --message "..."` |
| Missing --session flag | Need `--session main/isolated/custom` |
| Wrong delivery syntax | Use `--announce --channel <name> --to <id>` |
| Invented session types | Only main/isolated/custom sessions exist |
| Wrong storage location | Jobs at `~/.openclaw/cron/jobs.json` |

### Research Source
- Research document: `openclaw-101-research/RESEARCH.md` (lines 313-370)
- Official docs: https://docs.openclaw.ai/cli/cron
- Official docs: https://docs.openclaw.ai/automation/cron-jobs

### Tasks
- [ ] Fix cron add syntax completely:
  ```bash
  openclaw cron add \
    --name "job-name" \
    --cron "0 9 * * *" \
    --message "What to do" \
    --session isolated \
    --announce \
    --channel telegram \
    --to "123456789"
  ```
- [ ] Document session types:
  - `main`: Run during heartbeat, silent tasks
  - `isolated`: Fresh session per run (default for agentTurn)
  - `session:custom-id`: Persistent named session
- [ ] Add one-shot jobs with `--at` flag
- [ ] Document retry policy: 30s → 1m → 5m → 15m → 60m
- [ ] Fix run command: `openclaw cron run <job-id>` returns immediately
- [ ] Add checking runs: `openclaw cron runs --id <job-id>`
- [ ] Document storage: `~/.openclaw/cron/jobs.json`
- [ ] Add editing jobs: `openclaw cron edit <job-id>`
- [ ] Remove invented concepts:
  - No `openclaw cron add --name "..." --command "..."`
  - No custom session commands
  - No `cron_session_types` as commands

### Acceptance Criteria
- [ ] All cron examples use correct flag syntax
- [ ] Session types documented correctly
- [ ] Retry policy explained
- [ ] Storage location accurate
- [ ] No invented commands or concepts

### Priority
🔴 **Critical** - Core automation feature

### Estimated Effort
Medium (3-4 hours)

---
**Created:** 2026-04-01
**Depends on:** Research completed in `openclaw-101-research/RESEARCH.md`
