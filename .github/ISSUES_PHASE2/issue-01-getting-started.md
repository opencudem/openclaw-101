## [Phase 2] Fix 01-getting-started: Correct onboarding and gateway commands

### Problem
The getting started module contains several invented commands that don't exist in OpenClaw CLI, including `openclaw init`, `openclaw gateway start`, and references to port 3000 instead of 18789.

### Current Issues
| Current (Wrong) | Correct (Official) |
|-------------------|-------------------|
| `openclaw init` | `openclaw onboard --install-daemon` |
| `openclaw gateway start` | `openclaw gateway` or `openclaw gateway run` |
| Port 3000 | Port **18789** |
| Missing onboarding flags | Need --flow, --mode, --non-interactive |

### Research Source
- Research document: `openclaw-101-research/RESEARCH.md` (lines 25-65)
- Official docs: https://docs.openclaw.ai/cli/onboard
- Official docs: https://docs.openclaw.ai/cli/gateway

### Tasks
- [ ] Replace `openclaw init` with `openclaw onboard --install-daemon`
- [ ] Add onboarding flags documentation:
  - `--install-daemon`: Install system service
  - `--flow quickstart|manual`: Choose prompt level  
  - `--mode local|remote`: Gateway location
  - `--non-interactive`: For automation/scripts
  - `--accept-risk`: Acknowledge security
- [ ] Fix gateway commands section
  - `openclaw gateway` (foreground)
  - `openclaw gateway run` (alias)
  - `openclaw gateway status` with --require-rpc flag
  - `openclaw gateway probe` for debugging
- [ ] Correct port: Change all 3000 → 18789
- [ ] Verify dashboard command (`openclaw dashboard`) is correct
- [ ] Add verification steps
- [ ] Test installation steps if possible

### Acceptance Criteria
- [ ] No `openclaw init` remains in the guide
- [ ] Port 18789 used consistently
- [ ] Gateway commands match official docs
- [ ] Onboarding process is accurate
- [ ] Examples are copy-paste ready

### Priority
🔴 **Critical** - This is the first module users see

### Estimated Effort
Medium (2-3 hours)

---
**Created:** 2026-04-01
**Depends on:** Research completed in `openclaw-101-research/RESEARCH.md`
