## [Phase 2] Fix 04-skills: Correct skill installation and AgentSkills format

### Problem
Skills module has incorrect commands (`skill` instead of `skills`) and incomplete AgentSkills format documentation. Missing metadata.openclaw requirements.

### Current Issues
| Current (Wrong) | Correct (Official) |
|-------------------|-------------------|
| `openclaw skill install` | `openclaw skills install <slug>` |
| `openclaw skill list` | `openclaw skills list` |
| `openclaw skill info` | `openclaw skills info <name>` |
| Basic SKILL.md only | Missing metadata.openclaw format |
| No skill precedence docs | Need precedence order |

### Research Source
- Research document: `openclaw-101-research/RESEARCH.md` (lines 223-275)
- Official docs: https://docs.openclaw.ai/cli/skills
- Official docs: https://docs.openclaw.ai/tools/skills
- Official docs: https://docs.openclaw.ai/tools/clawhub

### Tasks
- [ ] Fix all `openclaw skill` → `openclaw skills` (plural)
- [ ] Document correct SKILL.md format:
  ```yaml
  ---
  name: skill-name
  description: Description here
  metadata:
    { "openclaw": { "requires": { "bins": ["uv"], "env": ["API_KEY"], "config": ["key.path"] }, "primaryEnv": "API_KEY" } }
  ---
  ```
- [ ] Add gating documentation:
  - `requires.bins`: binaries on PATH
  - `requires.env`: environment variables
  - `requires.config`: config paths
  - `requires.anyBins`: at least one binary
  - `always: true`: skip gates
- [ ] Document skill precedence (highest to lowest):
  1. `<workspace>/skills`
  2. `<workspace>/.agents/skills`
  3. `~/.agents/skills`
  4. `~/.openclaw/skills`
  5. Bundled skills
  6. `skills.load.extraDirs`
- [ ] Add missing commands:
  - `openclaw skills search "query"`
  - `openclaw skills list --eligible`
  - `openclaw skills check`
  - `clawhub sync --all`
- [ ] Document ClawHub CLI for publishing
- [ ] Add security notes (skills are untrusted code)

### Acceptance Criteria
- [ ] All commands use `skills` (plural)
- [ ] SKILL.md format is complete with metadata
- [ ] Skill precedence documented
- [ ] ClawHub integration explained
- [ ] No invented commands remain

### Priority
🔴 **Critical** - Core extensibility module

### Estimated Effort
Medium (3-4 hours)

---
**Created:** 2026-04-01
**Depends on:** Research completed in `openclaw-101-research/RESEARCH.md`
