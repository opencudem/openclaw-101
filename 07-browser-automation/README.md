# 07 - Browser Automation

> Automate web research, form filling, and data extraction with OpenClaw's browser skills.

## What this module solves

Browser automation lets OpenClaw interact with websites like a human: navigate pages, fill forms, click buttons, and extract data — all through natural language commands.

## When to use this module

- You need to research topics across multiple sites
- You want to automate form submissions
- You need to extract data from websites
- You want to monitor website changes

## Quick Start

### Install Browser Skill

```bash
# Install from ClawHub
openclaw skills install browser-automation

# Check browser status
openclaw browser status
```

### Basic Usage

```
User: Go to example.com and find the pricing
Agent: [Opens browser, navigates, extracts pricing info]

User: Search for "OpenClaw tutorials" and summarize the first 3 results
Agent: [Performs search, visits pages, extracts content, summarizes]

User: Fill out the contact form on example.com with my info
Agent: [Navigates to form, fills fields using your memory, submits]
```

## Browser CLI Commands

| Command | Description | Example |
|---------|-------------|---------|
| `openclaw browser status` | Check browser status | `openclaw browser status` |
| `openclaw browser start` | Start browser session | `openclaw browser start` |
| `openclaw browser stop` | Stop browser session | `openclaw browser stop` |
| `openclaw browser tabs` | List open tabs | `openclaw browser tabs` |
| `openclaw browser open <url>` | Open URL in new tab | `openclaw browser open https://example.com` |
| `openclaw browser close <tab-id>` | Close specific tab | `openclaw browser close tab-123` |
| `openclaw browser navigate <url>` | Navigate to URL | `openclaw browser navigate https://example.com` |
| `openclaw browser screenshot` | Take screenshot | `openclaw browser screenshot` |
| `openclaw browser snapshot` | Capture page state | `openclaw browser snapshot` |
| `openclaw browser click <selector>` | Click element | `openclaw browser click "button.submit"` |
| `openclaw browser type <selector> <text>` | Type into input | `openclaw browser type "#email" "user@example.com"` |
| `openclaw browser fill <selector> <value>` | Fill form field | `openclaw browser fill "#country" "Vietnam"` |
| `openclaw browser press <key>` | Press key | `openclaw browser press "Enter"` |
| `openclaw browser wait <ms>` | Wait milliseconds | `openclaw browser wait 1000` |
| `openclaw browser evaluate <script>` | Execute JavaScript | `openclaw browser evaluate "document.title"` |

## Copy-paste examples

### Research workflow via chat commands

Instead of writing JavaScript, use natural language with your agent:

```
User: Open browser and navigate to google.com
Agent: [Uses openclaw browser navigate]

User: Search for "OpenClaw tutorials" and visit the first 3 results
Agent: [Performs search, navigates, extracts content]

User: Extract all links from the current page
Agent: [Uses openclaw browser extractAll]
```

### Form automation via chat

```
User: Navigate to example.com/contact
Agent: [Opens contact page]

User: Fill the name field with "John", email with "john@example.com", and message with "Hello"
Agent: [Uses openclaw browser type for each field]

User: Click the submit button
Agent: [Uses openclaw browser click]

User: Take a screenshot to confirm
Agent: [Uses openclaw browser screenshot]
```

### Using browser in skills (via exec)

Skills can trigger browser commands via the exec tool:

```javascript
// ~/.openclaw/skills/browser-helper/scripts/navigate.js
const { execSync } = require('child_process');

module.exports = {
  async navigateTo({ url }) {
    // Use openclaw CLI to navigate
    const result = execSync(`openclaw browser navigate "${url}"`, { encoding: 'utf8' });
    return { success: true, result };
  },
  
  async takeScreenshot() {
    const result = execSync('openclaw browser screenshot', { encoding: 'utf8' });
    return { screenshot: result };
  }
};
```

### Price monitoring via chat

```
User: Navigate to example.com/product and extract the price
Agent: [Uses openclaw browser navigate and extract]

User: Compare with last check and alert if below $50
Agent: [Maintains history, calculates change, sends alert if needed]
```

Or create a cron job to check regularly:

```bash
openclaw cron add \
  --name "price-check" \
  --cron "0 */6 * * *" \
  --message "Check price on example.com/product and alert if below $50" \
  --announce \
  --channel telegram
```

## Safety Considerations

⚠️ **Browser automation is powerful — use responsibly:**

1. **Rate limiting**: Don't hammer websites. Add delays between requests.
2. **Terms of service**: Respect robots.txt and site terms.
3. **Sensitive data**: Never automate logins to financial/banking sites.
4. **DM policies**: For important actions, send to a restricted channel for approval

## Common mistakes and troubleshooting

| Problem | Solution |
|---------|----------|
| Page won't load | Check URL; verify site isn't blocking automation |
| Element not found | Wait for page load; check selector; add delay |
| CAPTCHA triggered | Pause for manual solve; don't automate CAPTCHA |
| Session expired | Re-authenticate; use saved session if available |

## Advanced patterns

### Using browser with cron

Schedule regular browser tasks:

```bash
# Daily price check
openclaw cron add \
  --name "daily-price-check" \
  --cron "0 9 * * *" \
  --message "Check prices on monitored sites and report changes" \
  --announce \
  --channel telegram
```

### Multi-step browser workflows

Chain multiple browser commands:

```
User: Navigate to dashboard.example.com, extract the metrics, and email the summary
Agent: [Uses browser navigate, extract, then email skill]
```

## Related modules and next step

- Previous: [06-automation](../06-automation/)
- Next: [08-workflows](../08-workflows/)
- Safety: [Approval patterns](../09-security-and-ops/approval-patterns.md)

---

**Time to complete:** 60 minutes  
**Prerequisites:** [06-automation](../06-automation/)  
**Outcome:** Web research and automation capabilities, data extraction workflows
