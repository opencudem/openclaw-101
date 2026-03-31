# Morning Briefing Workflow

> Start your day with weather, calendar, and priorities — automatically delivered.

## What it does

Every morning at 8 AM, receive a personalized briefing with:
- 🌤️ Current weather
- 📅 Today's calendar events
- ✅ Pending tasks
- 📰 Optional: News headlines

## Components Used

| Component | Purpose |
|-----------|---------|
| **Cron** | Triggers daily at 8 AM |
| **Weather skill** | Gets current conditions |
| **Calendar skill** | Fetches today's events |
| **Todo skill** | Lists pending tasks |
| **Telegram/Discord** | Delivers the briefing |

## Setup

### 1. Install Required Skills

```bash
openclaw skill install weather
openclaw skill install calendar
openclaw skill install todo
```

### 2. Configure Skills

```bash
# Set your location for weather
openclaw config set weather.default_location "Ho Chi Minh City"

# Configure calendar (Google Calendar example)
openclaw config set calendar.provider google
openclaw config set calendar.credentials ~/.openclaw/secrets/gcal.json
```

### 3. Create the Workflow

Create `~/.openclaw/skills/morning-briefing/scripts/briefing.js`:

```javascript
module.exports = {
  async generate() {
    // Gather data
    const weather = await skills.weather.getCurrent({
      location: config.weather?.default_location || 'local'
    });
    
    const calendar = await skills.calendar.getToday();
    const tasks = await skills.todo.getPending?.() || { count: 0 };
    
    // Format message
    const greeting = weather.temp > 20 ? 'Good morning! ☀️' : 'Good morning! 🧥';
    
    const eventsList = calendar.events?.length 
      ? calendar.events.map(e => `• ${e.time || 'All day'}: ${e.title}`).join('\n')
      : 'No events scheduled';
    
    const message = `${greeting}

🌤️ **Weather:** ${weather.condition}, ${weather.temp}°C

📅 **Today's Events:**
${eventsList}

✅ **Pending Tasks:** ${tasks.count || 0}

Have a great day! 🚀`;
    
    return message;
  },
  
  async send(channel = 'telegram') {
    const briefing = await this.generate();
    
    await skills.notify.send({
      channel,
      message: briefing
    });
    
    return 'Morning briefing sent!';
  }
};
```

### 4. Add SKILL.md

Create `~/.openclaw/skills/morning-briefing/SKILL.md`:

```markdown
# Morning Briefing

## Description
Generates and sends a personalized morning briefing.

## Tools

### generate
Create the briefing content.

**Returns:** Formatted message string

### send
Send briefing to a channel.

**Parameters:**
- channel (string): Target channel (default: telegram)

**Example:**
```javascript
send({ channel: 'telegram' })
```
```

### 5. Set Up Cron Job

```bash
openclaw cron add "morning-briefing" \
  --schedule "0 8 * * *" \
  --command "run skill morning-briefing send telegram" \
  --channel telegram
```

## Usage

### Automatic (via Cron)

The briefing arrives automatically at 8 AM daily.

### Manual

Trigger anytime by messaging your agent:

```
User: Send morning briefing
Agent: [Generates and sends briefing]
```

Or run directly:

```bash
openclaw skill run morning-briefing send telegram
```

## Customization

### Change Time

```bash
# 7 AM instead of 8 AM
openclaw cron update "morning-briefing" \
  --schedule "0 7 * * *"
```

### Add News Headlines

Add to `generate()`:

```javascript
const news = await skills.news.getHeadlines({ category: 'technology', limit: 3 });
const newsList = news.map(n => `• ${n.title}`).join('\n');

// Add to message:
📰 **Tech Headlines:**
${newsList}
```

### Weekend vs Weekday

```javascript
const day = new Date().getDay();
const isWeekend = day === 0 || day === 6;

if (isWeekend) {
  // Different briefing for weekends
  return this.generateWeekendBriefing();
}
```

### Multiple Channels

Send to different channels on different days:

```javascript
async sendMultiChannel() {
  const day = new Date().getDay();
  
  // Weekdays: Telegram (personal)
  if (day >= 1 && day <= 5) {
    await this.send('telegram');
  }
  
  // Weekends: Discord (shared with family)
  else {
    await this.send('discord');
  }
}
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Briefing not sending | Check `openclaw cron list` and gateway status |
| Weather not showing | Verify location in config |
| Calendar empty | Check calendar credentials |
| Wrong time | Check timezone with `echo $TZ` |

## Related

- [06-automation](../../06-automation/) - Cron setup
- [03-memory](../../03-memory/) - Personalization
- [CATALOG.md](../../CATALOG.md) - More skills

---

**Difficulty:** 🟢 Easy  
**Setup time:** 20 minutes  
**Maintenance:** Low
