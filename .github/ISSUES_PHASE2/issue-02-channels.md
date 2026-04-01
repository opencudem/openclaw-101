## [Phase 2] Fix 02-channels: Correct channel command syntax

### Problem
Channel commands use wrong syntax throughout the module. OpenClaw uses `channels` (plural) with `--channel` flag, not `channel add <name>`.

### Current Issues
| Current (Wrong) | Correct (Official) |
|-------------------|-------------------|
| `openclaw channel add telegram` | `openclaw channels add --channel telegram --token <token>` |
| `openclaw channel list` | `openclaw channels list` |
| `openclaw channel status` | `openclaw channels status` |
| `openclaw channel pause/resume` | Commands don't exist - use config or remove |

### Research Source
- Research document: `openclaw-101-research/RESEARCH.md` (lines 118-155)
- Official docs: https://docs.openclaw.ai/cli/channels
- Official docs: https://docs.openclaw.ai/channels/telegram
- Official docs: https://docs.openclaw.ai/channels/discord
- Official docs: https://docs.openclaw.ai/channels/whatsapp
- Official docs: https://docs.openclaw.ai/channels/slack

### Tasks
- [ ] Fix all `openclaw channel` → `openclaw channels`
- [ ] Add `--channel` and `--token` flags to all examples
- [ ] Update Telegram section:
  - Get token from @BotFather
  - Configure botToken in openclaw.json
  - DM policies: pairing, allowlist, open, disabled
  - Pairing commands: `openclaw pairing list/approve telegram <code>`
- [ ] Update Discord section:
  - Enable intents (Message Content required)
  - App Token + Bot Token
  - OAuth2 URL generation
  - SecretRef for token storage
- [ ] Add WhatsApp section:
  - QR pairing: `openclaw channels login --channel whatsapp`
  - allowFrom with E.164 phone numbers
  - groupPolicy and groupAllowFrom
- [ ] Add Slack section:
  - Socket Mode (default) vs HTTP mode
  - App Token (xapp-...) + Bot Token (xoxb-...)
  - Event subscriptions
  - Scopes checklist
- [ ] Document DM policies properly
- [ ] Remove `openclaw channel pause/resume` (doesn't exist)
- [ ] Add `openclaw channels capabilities` command

### Acceptance Criteria
- [ ] All commands use `channels` (plural)
- [ ] All examples have required flags
- [ ] Each channel has accurate setup steps
- [ ] Pairing flow documented for applicable channels
- [ ] No invented commands remain

### Priority
🔴 **Critical** - Core functionality module

### Estimated Effort
Large (4-6 hours) - Multiple channels to document

---
**Created:** 2026-04-01
**Depends on:** Research completed in `openclaw-101-research/RESEARCH.md`
