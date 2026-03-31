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
openclaw skill install browser-automation

# Or via npx for latest version
npx openclaw-skill-browser-automation
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

## Browser Actions

| Action | Description | Example |
|--------|-------------|---------|
| `navigate` | Go to URL | `navigate("https://example.com")` |
| `click` | Click element | `click("button.submit")` |
| `type` | Fill input | `type("#email", "user@example.com")` |
| `select` | Select dropdown | `select("#country", "Vietnam")` |
| `extract` | Get data | `extract(".price")` |
| `screenshot` | Capture page | `screenshot()` |
| `scroll` | Scroll page | `scroll("down")` |

## Copy-paste examples

### Research workflow

```javascript
// ~/.openclaw/skills/research/scripts/web-research.js
module.exports = {
  async researchTopic({ query, sources = 3 }) {
    // Search
    await browser.navigate(`https://google.com/search?q=${encodeURIComponent(query)}`);
    
    // Get results
    const links = await browser.extractAll('.g a[href]', 'href');
    const topLinks = links.slice(0, sources);
    
    // Visit and extract
    const results = [];
    for (const link of topLinks) {
      await browser.navigate(link);
      const content = await browser.extract('article, .content, main');
      results.push({ url: link, content });
    }
    
    return results;
  }
};
```

### Form automation

```javascript
// ~/.openclaw/skills/forms/scripts/fill-form.js
module.exports = {
  async fillContactForm({ name, email, message }) {
    await browser.navigate('https://example.com/contact');
    
    // Fill fields
    await browser.type('input[name="name"]', name);
    await browser.type('input[name="email"]', email);
    await browser.type('textarea[name="message"]', message);
    
    // Submit
    await browser.click('button[type="submit"]');
    
    // Confirm
    const confirmation = await browser.extract('.confirmation');
    return confirmation;
  }
};
```

### Price monitoring

```javascript
// ~/.openclaw/skills/monitor/scripts/price-check.js
const fs = require('fs').promises;
const path = require('path');

module.exports = {
  async checkPrice({ url, selector, threshold }) {
    await browser.navigate(url);
    const priceText = await browser.extract(selector);
    const price = parseFloat(priceText.replace(/[^0-9.]/g, ''));
    
    // Compare with previous
    const historyPath = path.join(process.env.HOME, '.openclaw/data/prices.json');
    let history = {};
    try {
      history = JSON.parse(await fs.readFile(historyPath, 'utf8'));
    } catch {}
    
    const previous = history[url];
    const change = previous ? ((price - previous) / previous * 100).toFixed(1) : 0;
    
    // Save new price
    history[url] = price;
    await fs.writeFile(historyPath, JSON.stringify(history, null, 2));
    
    return {
      price,
      previous,
      change: `${change}%`,
      alert: price < threshold
    };
  }
};
```

## Safety Considerations

⚠️ **Browser automation is powerful — use responsibly:**

1. **Rate limiting**: Don't hammer websites. Add delays between requests.
2. **Terms of service**: Respect robots.txt and site terms.
3. **Sensitive data**: Never automate logins to financial/banking sites.
4. **Approval gating**: For important actions, require confirmation:

```javascript
// Require approval before form submission
await skills.approval.request({
  message: 'Submit form with these details?',
  details: { name, email, message }
});
```

## Common mistakes and troubleshooting

| Problem | Solution |
|---------|----------|
| Page won't load | Check URL; verify site isn't blocking automation |
| Element not found | Wait for page load; check selector; add delay |
| CAPTCHA triggered | Pause for manual solve; don't automate CAPTCHA |
| Session expired | Re-authenticate; use saved session if available |

## Advanced patterns

### Session persistence

```javascript
// Save and restore login sessions
const fs = require('fs').promises;

module.exports = {
  async saveSession({ site }) {
    const cookies = await browser.getCookies();
    await fs.writeFile(
      `~/.openclaw/sessions/${site}.json`,
      JSON.stringify(cookies)
    );
  },
  
  async restoreSession({ site }) {
    const cookies = JSON.parse(
      await fs.readFile(`~/.openclaw/sessions/${site}.json`)
    );
    await browser.setCookies(cookies);
  }
};
```

### Parallel browsing

```javascript
// Check multiple sites concurrently
async function checkMultipleSites(urls) {
  const promises = urls.map(url => 
    browser.newContext().then(ctx => 
      ctx.navigate(url).then(() => ctx.extract('h1'))
    )
  );
  return Promise.all(promises);
}
```

## Related modules and next step

- Previous: [06-automation](../06-automation/)
- Next: [08-workflows](../08-workflows/)
- Safety: [Approval patterns](../09-security-and-ops/approval-patterns.md)

---

**Time to complete:** 60 minutes  
**Prerequisites:** [06-automation](../06-automation/)  
**Outcome:** Web research and automation capabilities, data extraction workflows
