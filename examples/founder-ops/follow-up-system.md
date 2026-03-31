# Follow-up System Workflow

> Never forget to follow up again. Track conversations, commitments, and reminders across all your channels.

## What it does

Maintains a follow-up queue:
- 📝 Notes things you promised to do
- 👥 Tracks who promised you things
- ⏰ Reminds you when to follow up
- 📊 Shows all pending items at a glance

## Components Used

| Component | Purpose |
|-----------|---------|
| **Memory** | Stores follow-up items |
| **Cron** | Daily reminder check |
| **Channels** | Receive reminders |
| **Natural language** | Parse "follow up with..." |

## Setup

### 1. Create the Skill

Create `~/.openclaw/skills/follow-up/scripts/manage.js`:

```javascript
const fs = require('fs').promises;
const path = require('path');

const DATA_FILE = path.join(
  process.env.HOME, 
  '.openclaw/memory/follow-ups.json'
);

module.exports = {
  // Add a new follow-up
  async add({ who, what, when, channel = 'telegram' }) {
    const data = await this.loadData();
    
    const item = {
      id: Date.now().toString(),
      who,
      what,
      when: new Date(when).toISOString(),
      channel,
      status: 'pending',
      created: new Date().toISOString(),
      reminded: false
    };
    
    data.items.push(item);
    await this.saveData(data);
    
    return `Added: Follow up with ${who} about "${what}" on ${when}`;
  },
  
  // List all pending follow-ups
  async list({ status = 'pending' } = {}) {
    const data = await this.loadData();
    const items = data.items.filter(i => i.status === status);
    
    if (items.length === 0) {
      return 'No pending follow-ups! 🎉';
    }
    
    const sorted = items.sort((a, b) => 
      new Date(a.when) - new Date(b.when)
    );
    
    return sorted.map(i => {
      const date = new Date(i.when).toLocaleDateString();
      const overdue = new Date(i.when) < new Date() ? '⚠️ OVERDUE ' : '';
      return `${overdue}[${date}] ${i.who}: "${i.what}"`;
    }).join('\n');
  },
  
  // Mark as complete
  async complete({ id }) {
    const data = await this.loadData();
    const item = data.items.find(i => i.id === id);
    
    if (!item) {
      return 'Follow-up not found';
    }
    
    item.status = 'completed';
    item.completed = new Date().toISOString();
    await this.saveData(data);
    
    return `✅ Completed: ${item.who} - "${item.what}"`;
  },
  
  // Check for reminders
  async checkReminders() {
    const data = await this.loadData();
    const now = new Date();
    const tomorrow = new Date(now.getTime() + 24 * 60 * 60 * 1000);
    
    const reminders = data.items.filter(i => {
      if (i.status !== 'pending') return false;
      const when = new Date(i.when);
      // Remind if due today or overdue
      return when <= tomorrow && !i.reminded;
    });
    
    for (const item of reminders) {
      await skills.notify.send({
        channel: item.channel,
        message: this.formatReminder(item)
      });
      
      item.reminded = true;
    }
    
    await this.saveData(data);
    return `Sent ${reminders.length} reminder(s)`;
  },
  
  formatReminder(item) {
    const when = new Date(item.when);
    const isOverdue = when < new Date();
    const urgency = isOverdue ? '⚠️ OVERDUE' : '⏰ REMINDER';
    
    return `${urgency}: Follow up with ${item.who}

"${item.what}"

Due: ${when.toLocaleDateString()}

Reply "complete ${item.id}" when done.`;
  },
  
  // Load/save data
  async loadData() {
    try {
      const content = await fs.readFile(DATA_FILE, 'utf8');
      return JSON.parse(content);
    } catch {
      return { items: [] };
    }
  },
  
  async saveData(data) {
    await fs.mkdir(path.dirname(DATA_FILE), { recursive: true });
    await fs.writeFile(DATA_FILE, JSON.stringify(data, null, 2));
  }
};
```

### 2. Add SKILL.md

Create `~/.openclaw/skills/follow-up/SKILL.md`:

```markdown
# Follow-up System

## Description
Track conversations and commitments requiring follow-up.

## Tools

### add
Add a new follow-up item.

**Parameters:**
- who (string): Person to follow up with
- what (string): What to follow up about
- when (string): When to follow up (date)
- channel (string): Where to send reminder

**Example:**
```javascript
add({
  who: "Alice",
  what: "Project proposal feedback",
  when: "2026-04-05",
  channel: "telegram"
})
```

### list
Show pending follow-ups.

**Parameters:**
- status (string): Filter by status (default: pending)

### complete
Mark a follow-up as done.

**Parameters:**
- id (string): Follow-up ID to complete

### checkReminders
Send reminders for upcoming/overdue items.
```

### 3. Set Up Daily Reminders

```bash
# Check for reminders every morning
openclaw cron add "follow-up-reminders" \
  --schedule "0 9 * * *" \
  --command "run skill follow-up checkReminders"
```

### 4. Auto-Extract from Conversations

Create a listener skill that catches follow-up phrases:

```javascript
// ~/.openclaw/skills/follow-up/scripts/extract.js
const FOLLOW_UP_PATTERNS = [
  /(?:remind me to|i need to|follow up with|follow up on)\s+(.+?)(?:\s+on\s+(.+))?$/i,
  /(\w+)\s+(?:will|promised to)\s+(.+?)(?:\s+by\s+(.+))?$/i
];

module.exports = {
  async extractFromMessage(message, context) {
    for (const pattern of FOLLOW_UP_PATTERNS) {
      const match = message.match(pattern);
      if (match) {
        return {
          detected: true,
          who: match[1] || context.user,
          what: match[2] || match[1],
          when: match[3] || this.defaultFollowUpDate()
        };
      }
    }
    return { detected: false };
  },
  
  defaultFollowUpDate() {
    // Default: 3 days from now
    const date = new Date();
    date.setDate(date.getDate() + 3);
    return date.toISOString().split('T')[0];
  }
};
```

## Usage

### Add a Follow-up

```
User: Remind me to follow up with Bob about the contract on Friday
Agent: Added: Follow up with Bob about "the contract" on Friday

User: Alice promised to send the report by next Tuesday
Agent: Added: Follow up with Alice about "send the report" on Tuesday
```

### Check Pending Items

```
User: Show my follow-ups
Agent: 
[2026-04-02] Bob: "the contract" ⚠️ OVERDUE
[2026-04-05] Alice: "send the report"
[2026-04-10] Team: "Q2 planning meeting"
```

### Mark Complete

```
User: Complete follow-up with Bob
Agent: ✅ Completed: Bob - "the contract"

User: I followed up with Alice
Agent: ✅ Completed: Alice - "send the report"
```

## Customization

### Categories/Tags

Add tags to organize follow-ups:

```javascript
async add({ who, what, when, tags = [] }) {
  const item = {
    // ... existing fields
    tags,  // ['work', 'urgent', 'sales']
  };
  // ...
}

// Then filter
async listByTag(tag) {
  const data = await this.loadData();
  return data.items.filter(i => i.tags.includes(tag));
}
```

### Priority Levels

```javascript
const PRIORITY_EMOJI = {
  high: '🔴',
  medium: '🟡',
  low: '🟢'
};

// Sort by priority in list()
const sorted = items.sort((a, b) => {
  const priorityOrder = { high: 0, medium: 1, low: 2 };
  return priorityOrder[a.priority] - priorityOrder[b.priority];
});
```

### Recurring Follow-ups

```javascript
async scheduleRecurring({ who, what, frequency }) {
  // frequency: 'weekly', 'monthly'
  const dates = this.generateDates(frequency);
  
  for (const date of dates) {
    await this.add({ who, what, when: date });
  }
}
```

### Integration with Calendar

```javascript
// Automatically add to calendar
async addWithCalendar({ who, what, when }) {
  // Add to follow-up system
  const id = await this.add({ who, what, when });
  
  // Also add to calendar
  await skills.calendar.createEvent({
    title: `Follow up: ${who}`,
    description: what,
    date: when
  });
  
  return id;
}
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Reminders not sending | Check cron is running |
| Data lost | Back up `~/.openclaw/memory/follow-ups.json` |
| Wrong dates | Use ISO format (YYYY-MM-DD) |
| Too many reminders | Adjust reminder logic or reduce frequency |

## Related

- [03-memory](../../03-memory/) - Persistence patterns
- [06-automation](../../06-automation/) - Cron setup
- [05-integrations](../../05-integrations/) - Calendar integration

---

**Difficulty:** 🟡 Medium  
**Setup time:** 25 minutes  
**Maintenance:** Very Low (self-managing)
