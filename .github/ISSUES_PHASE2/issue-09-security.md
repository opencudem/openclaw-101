## [Phase 2] Fix 09-security-and-ops: Remove invented approval patterns

### Problem
Security module contains an entire invented approval system (`skills.approval.request`) that does not exist in OpenClaw. Need to document real security mechanisms.

### Current Issues
- **Invented**: `skills.approval.request` - This does not exist
- **Invented**: Multi-approver workflows - Not a native feature
- **Invented**: `skills.audit.log` - Not a native feature
- **Missing**: Real security: DM policies, pairing, `openclaw security audit`

### Research Source
- Research document: `openclaw-101-research/RESEARCH.md` (lines 400-420)
- Official docs: https://docs.openclaw.ai/gateway/security
- Official docs: https://docs.openclaw.ai/channels/pairing
- Official docs: https://docs.openclaw.ai/cli/security

### Tasks
- [ ] **REMOVE** all `skills.approval.request` examples
- [ ] **REMOVE** multi-approver workflow examples
- [ ] **REMOVE** `skills.audit.log` examples
- [ ] Document real security features:
  - DM policies (pairing, allowlist, open, disabled)
  - Group policies and allowFrom
  - Pairing system as security mechanism
  - `openclaw security audit` command
  - `openclaw security audit --deep`
  - `openclaw security audit --fix`
- [ ] Document SecretRef for secure credentials
- [ ] Document `openclaw doctor --fix` for security repairs
- [ ] Document sandboxing: `agents.defaults.sandbox.mode`
- [ ] Document channel-specific allowlists
- [ ] Rewrite "Approval Patterns" to use real mechanisms:
  - Use DM policies for access control
  - Use pairing for user verification
  - Use `openclaw doctor` for diagnostics

### Acceptance Criteria
- [ ] No invented `skills.approval` code remains
- [ ] Real OpenClaw security documented
- [ ] DM/group policies explained
- [ ] Pairing documented as security feature
- [ ] `openclaw security audit` included

### Priority
🔴 **Critical** - Security misinformation is dangerous

### Estimated Effort
Large (4-5 hours) - Requires complete rewrite

---
**Created:** 2026-04-01
**Depends on:** Research completed in `openclaw-101-research/RESEARCH.md`
