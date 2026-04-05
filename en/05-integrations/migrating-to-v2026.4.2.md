# Migrating to OpenClaw v2026.4.2

> Breaking changes: xAI and Firecrawl plugin configuration paths have moved. Use this guide to migrate without downtime.

---

## What Changed

OpenClaw v2026.4.2 enforces a strict boundary between core system configuration and plugin-specific settings. Previously, plugin configs lived in the core namespace (`tools.web.*`). Now they live in dedicated plugin namespaces (`plugins.entries.*`).

This is a **breaking change** — legacy config paths are silently ignored after upgrade.

### Affected Features

| Feature | Old Path | New Path | Auth Key |
|---------|----------|----------|----------|
| xAI Search (Grok) | `tools.web.x_search.*` | `plugins.entries.xai.config.xSearch.*` | `plugins.entries.xai.config.webSearch.apiKey` |
| Firecrawl Web Fetch | `tools.web.fetch.firecrawl.*` | `plugins.entries.firecrawl.config.webFetch.*` | `plugins.entries.firecrawl.config.apiKey` |

---

## Before You Upgrade

### 1. Backup Your Config

```bash
# Backup current configuration
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup.$(date +%Y%m%d)
```

### 2. Check if You're Affected

```bash
# Check for legacy xAI config
grep -q "tools.web.x_search" ~/.openclaw/openclaw.json && echo "xAI migration needed"

# Check for legacy Firecrawl config
grep -q "tools.web.fetch.firecrawl" ~/.openclaw/openclaw.json && echo "Firecrawl migration needed"
```

---

## Migration Steps

### Option 1: Automated Migration (Recommended)

```bash
# Upgrade OpenClaw first
npm install -g openclaw@latest

# Run the automated fix
openclaw doctor --fix
```

The `doctor --fix` command:
- Detects legacy config paths
- Migrates values to new plugin namespaces
- Backs up the original config
- Validates the new configuration

### Option 2: Manual Migration

If automated migration fails or you prefer control:

**Old xAI Config:**
```json5
{
  tools: {
    web: {
      x_search: {
        enabled: true,
        apiKey: "xai-...",
        model: "grok-2"
      }
    }
  }
}
```

**New xAI Config:**
```json5
{
  plugins: {
    entries: {
      xai: {
        config: {
          xSearch: {
            enabled: true,
            model: "grok-2"
          },
          webSearch: {
            apiKey: "xai-..."  // or use XAI_API_KEY env var
          }
        }
      }
    }
  }
}
```

**Old Firecrawl Config:**
```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "fc-...",
          baseUrl: "https://api.firecrawl.dev"
        }
      }
    }
  }
}
```

**New Firecrawl Config:**
```json5
{
  plugins: {
    entries: {
      firecrawl: {
        config: {
          webFetch: {
            apiKey: "fc-...",  // or use FIRECRAWL_API_KEY env var
            baseUrl: "https://api.firecrawl.dev"
          }
        }
      }
    }
  }
}
```

---

## Verify the Migration

### Test xAI Search

```bash
# Test web search with Grok
openclaw tool web_search --query "OpenClaw v2026.4.2 changes" --provider grok
```

### Test Firecrawl Fetch

```bash
# Test web fetch with Firecrawl
openclaw tool web_fetch --url "https://docs.openclaw.ai" --provider firecrawl
```

### Check Plugin Status

```bash
# Verify plugins are loaded correctly
openclaw plugins list
```

---

## Troubleshooting

### "xAI search stopped working after upgrade — no error logs"

**Cause**: Legacy config path `tools.web.x_search` is silently ignored.

**Fix**: Run `openclaw doctor --fix` or manually migrate config.

### "Firecrawl fetch returns empty results"

**Cause**: Old config path not recognized.

**Fix**: Verify migration with:
```bash
openclaw config get plugins.entries.firecrawl.config.webFetch.apiKey
```

### Migration command fails

**Check**: Ensure OpenClaw is v2026.4.2 or later:
```bash
openclaw --version
```

**Fallback**: Use manual migration steps above.

---

## Environment Variable Support

You can still use environment variables instead of hardcoded API keys:

| Service | Environment Variable |
|---------|---------------------|
| xAI | `XAI_API_KEY` |
| Firecrawl | `FIRECRAWL_API_KEY` |

Example with env vars:
```json5
{
  plugins: {
    entries: {
      xai: {
        config: {
          xSearch: { enabled: true },
          webSearch: {
            // apiKey omitted — will use XAI_API_KEY env var
          }
        }
      }
    }
  }
}
```

---

## Post-Migration Checklist

- [ ] Backed up original config
- [ ] Ran `openclaw doctor --fix` (or manual equivalent)
- [ ] Verified xAI search works: `openclaw tool web_search --query test --provider grok`
- [ ] Verified Firecrawl fetch works (if using)
- [ ] Checked `openclaw plugins list` shows correct plugin status
- [ ] Removed old `tools.web.x_search` and `tools.web.fetch.firecrawl` entries from config
- [ ] Updated any documentation or runbooks referencing old paths

---

## Why This Changed

The plugin namespace separation:
- **Prevents core config bloat** as more plugins are added
- **Enables plugin-specific versioning** and settings
- **Makes ownership clear** — core owns `tools.web.*`, plugins own `plugins.entries.*`
- **Allows plugin-specific validation** without core system changes

---

## Related Documentation

- [Module 05: Integrations](./README.md)
- [OpenClaw Doctor](../../10-cli-and-reference/cli-reference.md#doctor)
- [Web Search Configuration](../../10-cli-and-reference/config-reference.md#web-search)
- [Official Migration Guide](https://www.xugj520.cn/en/archives/openclaw-2026-migration-configuration-security-task-flow.html)

---

**Last Updated**: April 4, 2026  
**Applies To**: OpenClaw v2026.4.2+  
**Breaking Change**: Yes — action required on upgrade
