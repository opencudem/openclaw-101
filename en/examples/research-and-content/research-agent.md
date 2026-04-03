# Research Agent Workflow

> Automated research assistant that searches, visits, extracts, and summarizes information from multiple sources.

## What it does

Given a research topic, the agent will:
1. 🔍 Search for relevant sources
2. 🌐 Visit top results
3. 📄 Extract key information
4. 📝 Summarize findings
5. 📧 Deliver formatted report

## Components Used

| Component | Purpose |
|-----------|---------|
| **Search skill** | Find relevant sources |
| **Browser automation** | Visit and extract content |
| **Summarizer skill** | Condense information |
| **Email/Notion** | Deliver final report |
| **Cron** | Scheduled research tasks |

## Setup

### 1. Install Required Skills

```bash
openclaw skill install search
openclaw skill install browser-automation
openclaw skill install summarizer
openclaw skill install email
```

### 2. Configure API Keys

```bash
# Search (e.g., Brave Search API)
openclaw config set search.api_key YOUR_BRAVE_API_KEY

# Email
openclaw config set email.provider agentmail
openclaw config set email.api_key YOUR_AGENTMAIL_KEY
```

### 3. Create the Research Agent

Create `~/.openclaw/skills/research-agent/scripts/agent.js`:

```javascript
module.exports = {
  async research({ 
    topic, 
    sources = 5, 
    depth = 'medium',  // 'quick', 'medium', 'deep'
    output = 'email',
    format = 'report'  // 'report', 'bullets', 'executive'
  }) {
    // 1. Search for sources
    console.log(`Searching for: ${topic}`);
    const searchResults = await skills.search.query({
      query: topic,
      limit: sources
    });
    
    // 2. Visit and extract from each source
    const sources = [];
    for (const result of searchResults) {
      try {
        console.log(`Visiting: ${result.url}`);
        
        await browser.navigate(result.url);
        
        // Wait for content to load
        await browser.waitFor('article, main, .content', 5000);
        
        // Extract based on depth
        const content = await this.extractContent(depth);
        
        sources.push({
          title: result.title,
          url: result.url,
          content: content,
          relevance: result.score
        });
        
        // Be nice to websites
        await this.delay(2000);
      } catch (error) {
        console.error(`Failed to extract from ${result.url}: ${error.message}`);
      }
    }
    
    // 3. Summarize each source
    const summaries = [];
    for (const source of sources) {
      const summary = await skills.summarizer.summarize({
        text: source.content,
        length: depth === 'quick' ? 'short' : 'medium'
      });
      
      summaries.push({
        ...source,
        summary
      });
    }
    
    // 4. Generate overall synthesis
    const synthesis = await this.synthesize(summaries, topic);
    
    // 5. Format and deliver
    const report = this.formatReport({
      topic,
      summaries,
      synthesis,
      format
    });
    
    await this.deliver(report, output);
    
    return {
      topic,
      sourcesChecked: sources.length,
      reportLength: report.length,
      delivered: true
    };
  },
  
  async extractContent(depth) {
    const selectors = {
      quick: 'h1, h2, p:first-of-type',
      medium: 'article, main, .content',
      deep: 'body'
    };
    
    const content = await browser.extract(selectors[depth]);
    
    // Clean up the content
    return content
      .replace(/\s+/g, ' ')
      .replace(/\n{3,}/g, '\n\n')
      .slice(0, depth === 'quick' ? 1000 : depth === 'medium' ? 5000 : 20000);
  },
  
  async synthesize(summaries, topic) {
    const combined = summaries
      .map(s => `Source: ${s.title}\n${s.summary}`)
      .join('\n\n---\n\n');
    
    return await skills.summarizer.synthesize({
      texts: summaries.map(s => s.summary),
      topic,
      focus: 'key_insights'
    });
  },
  
  formatReport({ topic, summaries, synthesis, format }) {
    const formats = {
      report: this.formatAsReport,
      bullets: this.formatAsBullets,
      executive: this.formatAsExecutive
    };
    
    return formats[format].call(this, { topic, summaries, synthesis });
  },
  
  formatAsReport({ topic, summaries, synthesis }) {
    const date = new Date().toLocaleDateString();
    
    let report = `# Research Report: ${topic}\n\n`;
    report += `Generated: ${date}\n\n`;
    report += `## Executive Summary\n\n${synthesis}\n\n`;
    report += `## Sources\n\n`;
    
    for (const source of summaries) {
      report += `### ${source.title}\n`;
      report += `[${source.url}](${source.url})\n\n`;
      report += `${source.summary}\n\n`;
    }
    
    return report;
  },
  
  formatAsBullets({ topic, summaries, synthesis }) {
    let report = `**${topic}**\n\n`;
    report += `Key Findings:\n`;
    report += synthesis.split('\n').map(l => `• ${l}`).join('\n');
    report += '\n\nSources:\n';
    
    for (const source of summaries) {
      report += `• [${source.title}](${source.url})\n`;
    }
    
    return report;
  },
  
  formatAsExecutive({ topic, synthesis }) {
    return `**Research Brief: ${topic}**\n\n${synthesis}\n\n_This is an AI-generated summary. Verify critical information._`;
  },
  
  async deliver(report, output) {
    switch (output) {
      case 'email':
        await skills.email.send({
          to: process.env.RESEARCH_EMAIL,
          subject: `Research Report: ${topic}`,
          body: report
        });
        break;
        
      case 'notion':
        await skills.notion.createPage({
          parent: process.env.NOTION_RESEARCH_DB,
          title: `Research: ${topic}`,
          content: report
        });
        break;
        
      case 'channel':
        await skills.notify.send({
          channel: 'telegram',
          message: report.slice(0, 4000)  // Telegram limit
        });
        break;
    }
  },
  
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
};
```

### 4. Add SKILL.md

Create `~/.openclaw/skills/research-agent/SKILL.md`:

```markdown
# Research Agent

## Description
Automated research assistant for gathering and synthesizing information.

## Tools

### research
Perform comprehensive research on a topic.

**Parameters:**
- topic (string): Research subject
- sources (number): Number of sources to check (default: 5)
- depth (string): Research depth - 'quick', 'medium', or 'deep'
- output (string): Output method - 'email', 'notion', or 'channel'
- format (string): Report format - 'report', 'bullets', or 'executive'

**Example:**
```javascript
research({
  topic: "OpenClaw vs Claude Desktop",
  sources: 8,
  depth: 'medium',
  output: 'email',
  format: 'report'
})
```

### Quick Research

Shorthand for quick research:
```
research topic quickly
```
```

## Usage

### Manual Research

```
User: Research "latest AI agent frameworks" and email me a report
Agent: [Conducts research and sends email]
```

### Deep Research

```
User: Deep research on "OpenClaw best practices" with 10 sources
Agent: [Extensive research with detailed report]
```

### Quick Check

```
User: Quick summary of "React 19 features"
Agent: [Fast search and brief summary]
```

## Customization

### Domain-Specific Research

```javascript
// Research only specific sites
async researchTech({ topic }) {
  const trustedSites = [
    'site:github.com',
    'site:stackoverflow.com',
    'site:docs.openclaw.ai'
  ];
  
  const query = `${topic} (${trustedSites.join(' OR ')})`;
  return this.research({ topic: query, sources: 10 });
}
```

### Scheduled Research

```bash
# Weekly industry roundup
openclaw cron add "weekly-research" \
  --schedule "0 9 * * 1" \
  --command "run skill research-agent research 'AI agents weekly news' --sources=10 --output=email"
```

### Comparison Research

```javascript
async compareResearch({ topics }) {
  const results = {};
  
  for (const topic of topics) {
    results[topic] = await this.research({ 
      topic, 
      sources: 5, 
      depth: 'quick' 
    });
  }
  
  // Generate comparison table
  return this.formatComparison(results);
}
```

## Safety & Ethics

⚠️ **Important considerations:**

1. **Rate limiting**: Add delays between requests
2. **Respect robots.txt**: Don't hammer websites
3. **Source attribution**: Always cite sources
4. **Accuracy disclaimer**: Note that summaries are AI-generated
5. **Copyright**: Don't reproduce full articles

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Search API errors | Check API key and quotas |
| Extraction fails | Try different selectors; check if site blocks automation |
| Report too long | Reduce depth or sources; chunk output |
| Timeout errors | Add longer delays; reduce concurrent requests |

## Related

- [07-browser-automation](../../07-browser-automation/) - Web automation
- [05-integrations](../../05-integrations/) - Notion, email
- [06-automation](../../06-automation/) - Scheduled research

---

**Difficulty:** 🔴 Advanced  
**Setup time:** 45 minutes  
**Maintenance:** Medium (API quotas, site changes)
