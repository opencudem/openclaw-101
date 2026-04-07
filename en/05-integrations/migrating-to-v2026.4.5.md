# Migrating to OpenClaw v2026.4.5

> Breaking changes: Legacy config aliases removed. Use this guide to migrate without downtime.

**Version Introduced:** v2026.4.5 (April 6, 2026)  
**Breaking Change:** Yes — action required on upgrade  
**Migration Command:** `openclaw doctor --fix`

---

## What Changed

OpenClaw v2026.4.5 removes legacy public config aliases in favor of canonical paths. While old configs still load (with warnings), the legacy paths are deprecated and will be removed in a future major release.

The `openclaw doctor --fix` command handles automatic migration.

### Affected Config Paths

| Old Path | New Path | Component |
|----------|----------|-----------|
| `talk.voiceId` | `plugins.entries.tts.config.voiceId` | TTS voice selection |
| `talk.apiKey` | `providers.tts.*.apiKey` | TTS provider API key |
| `agents.*.sandbox.perSession` | `agents.*.sandbox.strategy` | Sandbox strategy |
| `browser.ssrfPolicy.allowPrivateNetwork` | `browser.security.allowPrivateNetwork` | Browser SSRF policy |
| `hooks.internal.handlers` | `hooks.handlers` | Hook handlers |
| Channel/group/room `allow` toggles | `enabled` flag | Channel enablement |

---

## Before You Upgrade

### 1. Backup Your Config

```bash
# Backup current configuration with timestamp
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup.$(date +%Y%m%d)
```

### 2. Check if You're Affected

```bash
# Check for legacy config paths
grep -E "(talk\.(voiceId|apiKey)|perSession|ssrfPolicy|hooks\.internal)" ~/.openclaw/openclaw.json && echo "Migration needed"
```

---

## Migration Steps

### Option 1: Automated Migration (Recommended)

```bash
# Upgrade OpenClaw first
npm install -g openclaw@latest

# Verify version
openclaw --version

# Run the automated fix
openclaw doctor --fix
```

The `doctor --fix` command will:
- Detect legacy config aliases
- Migrate values to canonical paths
- Create a backup of the original config
- Validate the new configuration
- Report any issues that need manual attention

### Option 2: Manual Migration

If automated migration fails or you prefer control:

**Old TTS Config:**
```json5
{
  talk: {
    voiceId: "nova",
    apiKey: "sk-..."
  }
}
```

**New TTS Config:**
```json5
{
  plugins: {
    entries: {
      tts: {
        config: {
          voiceId: "nova"
        }
      }
    }
  },
  providers: {
    tts: {
      openai: {
        apiKey: "sk-..."  // or use OPENAI_API_KEY env var
      }
    }
  }
}
```

**Old Sandbox Config:**
```json5
{
  agents: {
    myAgent: {
      sandbox: {
        perSession: true
      }
    }
  }
}
```

**New Sandbox Config:**
```json5
{
  agents: {
    myAgent: {
      sandbox: {
        strategy: "perSession"  // or "shared", "isolated"
      }
    }
  }
}
```

**Old Browser SSRF Config:**
```json5
{
  browser: {
    ssrfPolicy: {
      allowPrivateNetwork: false
    }
  }
}
```

**New Browser SSRF Config:**
```json5
{
  browser: {
    security: {
      allowPrivateNetwork: false
    }
  }
}
```

**Old Hooks Config:**
```json5
{
  hooks: {
    internal: {
      handlers: ["default"]
    }
  }
}
```

**New Hooks Config:**
```json5
{
  hooks: {
    handlers: ["default"]
  }
}
```

**Old Channel Allow Toggles:**
```json5
{
  channels: {
    telegram: {
      groups: {
        myGroup: {
          allow: true
        }
      }
    }
  }
}
```

**New Channel Enabled Flag:**
```json5
{
  channels: {
    telegram: {
      groups: {
        myGroup: {
          enabled: true
        }
      }
    }
  }
}
```

---

## Verify the Migration

### Check Config Load

```bash
# Validate config loads without warnings
openclaw config validate
```

### Test TTS (if configured)

```bash
# Test TTS functionality
openclaw tool tts_speak --text "Migration complete" --provider openai
```

### Check Sandbox Strategy

```bash
# Verify agent sandbox configuration
openclaw config get agents.myAgent.sandbox.strategy
```

### Review Doctor Output

```bash
# Run doctor without --fix to see current status
openclaw doctor
```

---

## Troubleshooting

### "Config validation warnings after upgrade"

**Cause**: Legacy aliases still present in config.

**Fix**: Run `openclaw doctor --fix` or manually remove old paths.

### "TTS stopped working — no voice output"

**Cause**: `talk.voiceId` and `talk.apiKey` not migrated.

**Fix**: Verify migration with:
```bash
openclaw config get plugins.entries.tts.config.voiceId
openclaw config get providers.tts.openai.apiKey
```

### "Sandbox behavior changed unexpectedly"

**Cause**: `perSession` boolean migrated to `strategy` enum.

**Fix**: Check the strategy value:
```bash
openclaw config get agents.*.sandbox.strategy
```

Valid values: `"perSession"`, `"shared"`, `"isolated"`

### "Browser requests to private networks failing"

**Cause**: `ssrfPolicy` path changed to `security`.

**Fix**: Update browser config:
```json5
{
  browser: {
    security: {
      allowPrivateNetwork: true  // if needed for your use case
    }
  }
}
```

### Migration command fails

**Check**: Ensure OpenClaw is v2026.4.5 or later:
```bash
openclaw --version
```

**Fallback**: Use manual migration steps above.

---

## Environment Variable Support

All API keys can still use environment variables instead of hardcoded values:

| Config Path | Environment Variable |
|-------------|---------------------|
| `providers.tts.openai.apiKey` | `OPENAI_API_KEY` |
| `providers.tts.elevenlabs.apiKey` | `ELEVENLABS_API_KEY` |
| `providers.tts.google.apiKey` | `GOOGLE_API_KEY` |

Example with env vars:
```json5
{
  providers: {
    tts: {
      openai: {
        // apiKey omitted — will use OPENAI_API_KEY env var
      }
    }
  }
}
```

---

## Post-Migration Checklist

- [ ] Backed up original config before migration
- [ ] Ran `openclaw doctor --fix` (or manual equivalent)
- [ ] Verified `openclaw config validate` passes without warnings
- [ ] Tested TTS if previously configured
- [ ] Verified agent sandbox strategy values are correct
- [ ] Checked browser security settings if using browser tools
- [ ] Reviewed hook handler configuration
- [ ] Updated channel/group `allow` to `enabled` where applicable
- [ ] Updated any documentation referencing old config paths

---

## What's New in v2026.4.5

See the following new guides for features introduced in this release:

- [Video Generation Guide](../../08-workflows/video-generation.md) — AI video generation with xAI, Runway, and Model Studio
- [Music Generation Guide](../../08-workflows/music-generation.md) — AI music with Google Lyria, MiniMax, and ComfyUI
- [Amazon Bedrock Mantle](../../05-integrations/bedrock.md#mantle) — Inference profile discovery and auto-setup
- [SearXNG Provider](../../05-integrations/searxng.md) — Privacy-focused web search

---

## Why This Changed

Config path standardization:
- **Reduces confusion** — one canonical path per setting
- **Enables better validation** — each path has strict typing
- **Prepares for future features** — cleaner namespace for upcoming additions
- **Improves documentation clarity** — single source of truth for each config

The migration is a one-time process. Future releases will maintain backward compatibility with v2026.4.5+ config paths.

---

## Related Documentation

- [Module 05: Integrations](./README.md)
- [Module 08: Workflows](../../08-workflows/README.md)
- [OpenClaw Doctor](../../10-cli-and-reference/cli-reference.md#doctor)
- [TTS Configuration](../../10-cli-and-reference/config-reference.md#tts)
- [Sandbox Configuration](../../10-cli-and-reference/config-reference.md#sandbox)
- [Browser Configuration](../../10-cli-and-reference/config-reference.md#browser)
- [v2026.4.2 Migration](./migrating-to-v2026.4.2.md) — If upgrading from earlier versions

---

**Last Updated**: April 7, 2026  
**Applies To**: OpenClaw v2026.4.5+  
**Previous Migration**: [v2026.4.2](./migrating-to-v2026.4.2.md)
