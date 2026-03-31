# 09 - Security and Ops

> Secure your OpenClaw deployment with approvals, permissions, and reliability patterns.

## What this module solves

Powerful automation needs guardrails. This module covers permission systems, approval workflows, and operational best practices.

## When to use this module

- You're running OpenClaw in production
- You handle sensitive data
- You need audit trails
- You want approval gating for destructive actions

## Security Layers

```
┌─────────────────────────────────────────┐
│           PERIMETER                     │
│  - Channel restrictions                 │
│  - IP allowlisting                      │
├─────────────────────────────────────────┤
│           ACCESS CONTROL                │
│  - User authentication                  │
│  - Role-based permissions               │
├─────────────────────────────────────────┤
│           APPROVALS                     │
│  - Human-in-the-loop                    │
│  - Timeout handling                     │
├─────────────────────────────────────────┤
│           AUDIT                         │
│  - Action logging                       │
│  - Change tracking                      │
└─────────────────────────────────────────┘
```

## Quick Start

### Channel Restrictions

Limit which channels can execute sensitive commands:

```yaml
# ~/.openclaw/config.yaml
security:
  allowed_channels:
    - telegram:12345678      # Your private chat
    - discord:987654321      # Your private server
  
  sensitive_commands:
    - execute_shell
    - send_email
    - deploy
```

### Approval Gating

Require approval for critical actions:

```javascript
// ~/.openclaw/skills/secure-ops/scripts/deploy.js
module.exports = {
  async deploy({ environment, version }) {
    // Request approval
    const approval = await skills.approval.request({
      channel: 'telegram',
      message: `Deploy ${version} to ${environment}?`,
      timeout: '30m',
      approvers: ['admin']  // Only admins can approve
    });
    
    if (!approval) {
      return 'Deployment cancelled (no approval)';
    }
    
    // Proceed with deployment
    await this.runDeployment(environment, version);
    
    // Log the action
    await skills.audit.log({
      action: 'deploy',
      user: approval.approver,
      details: { environment, version }
    });
    
    return `Deployed ${version} to ${environment}`;
  }
};
```

## Copy-paste examples

### Permission system

```javascript
// ~/.openclaw/skills/permissions/scripts/check.js
const ROLES = {
  admin: ['*'],           // All permissions
  developer: ['read', 'write', 'execute'],
  viewer: ['read']
};

const PERMISSIONS = {
  'read': ['query', 'view', 'list'],
  'write': ['create', 'update', 'delete'],
  'execute': ['run', 'deploy', 'restart'],
  'admin': ['*']
};

module.exports = {
  async checkPermission({ user, action, resource }) {
    const userRole = await this.getUserRole(user);
    const allowed = ROLES[userRole] || [];
    
    // Check if role has permission
    for (const perm of allowed) {
      if (perm === '*') return true;
      if (PERMISSIONS[perm]?.includes(action)) return true;
    }
    
    return false;
  },
  
  async requirePermission({ user, action, resource }) {
    const hasPermission = await this.checkPermission({ user, action, resource });
    
    if (!hasPermission) {
      throw new Error(`User ${user} lacks permission to ${action} on ${resource}`);
    }
    
    return true;
  }
};
```

### Audit logging

```javascript
// ~/.openclaw/skills/audit/scripts/logger.js
const fs = require('fs').promises;
const path = require('path');

module.exports = {
  async log({ action, user, details, outcome }) {
    const entry = {
      timestamp: new Date().toISOString(),
      action,
      user,
      details,
      outcome,
      ip: this.getClientIP(),
      session: this.getSessionID()
    };
    
    // Write to audit log
    const logPath = path.join(
      process.env.HOME,
      '.openclaw/audit',
      `${new Date().toISOString().split('T')[0]}.log`
    );
    
    await fs.mkdir(path.dirname(logPath), { recursive: true });
    await fs.appendFile(logPath, JSON.stringify(entry) + '\n');
    
    // Also send to secure channel if sensitive
    if (this.isSensitive(action)) {
      await skills.notify.send({
        channel: 'audit-log',
        message: `[AUDIT] ${user} performed ${action}`
      });
    }
  }
};
```

### Secure session isolation

```javascript
// ~/.openclaw/skills/secure-ops/scripts/isolate.js
module.exports = {
  async runIsolated({ skill, args, timeout = 30000 }) {
    // Create isolated context
    const context = {
      allowedSkills: skill.allowedSkills || [],
      allowedChannels: skill.allowedChannels || [],
      maxDuration: timeout,
      readOnly: skill.readOnly || false
    };
    
    try {
      // Run with restrictions
      const result = await Promise.race([
        this.executeWithContext(skill, args, context),
        new Promise((_, reject) => 
          setTimeout(() => reject(new Error('Timeout')), timeout)
        )
      ]);
      
      return { success: true, result };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }
};
```

## Operational Patterns

### Health checks

```bash
# Add health check job
openclaw cron add "health-check" \
  --schedule "*/15 * * * *" \
  --command "check gateway health and report"
```

### Backup configuration

```bash
# Backup script for OpenClaw config
cat > ~/.openclaw/scripts/backup.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d)
BACKUP_DIR="$HOME/.openclaw/backups/$DATE"

mkdir -p "$BACKUP_DIR"
cp -r "$HOME/.openclaw/config" "$BACKUP_DIR/"
cp -r "$HOME/.openclaw/skills" "$BACKUP_DIR/"
cp -r "$HOME/.openclaw/memory" "$BACKUP_DIR/"

echo "Backup saved to $BACKUP_DIR"
EOF

chmod +x ~/.openclaw/scripts/backup.sh
```

### Recovery procedures

```markdown
# ~/.openclaw/RECOVERY.md

## Gateway Won't Start

1. Check logs: `openclaw gateway logs --tail 100`
2. Check port conflicts: `lsof -i :3000`
3. Reset config: `openclaw config reset`
4. Restore from backup: `cp -r ~/.openclaw/backups/latest/* ~/.openclaw/`

## Lost Channel Connection

1. Check token validity
2. Re-authenticate: `openclaw channel remove <name> && openclaw channel add <name>`
3. Verify gateway is running: `openclaw gateway status`

## Data Corruption

1. Stop gateway
2. Restore from last known good backup
3. Restart gateway
```

## Common mistakes and troubleshooting

| Problem | Solution |
|---------|----------|
| Unauthorized access | Review allowed_channels; enable auth |
| Approval fatigue | Use tiered approvals; batch requests |
| Missing audit logs | Check permissions on audit directory |
| Session leaks | Implement timeouts; clean up contexts |

## Advanced patterns

### Multi-signature approvals

```javascript
// Require multiple approvers for critical actions
module.exports = {
  async requireMultipleApprovals({ action, minApprovers = 2 }) {
    const approvals = [];
    const approvers = await this.getEligibleApprovers(action);
    
    for (const approver of approvers) {
      const approval = await skills.approval.request({
        user: approver,
        message: `Approve ${action}? (${approvals.length}/${minApprovers})`,
        timeout: '1h'
      });
      
      if (approval) {
        approvals.push(approver);
        if (approvals.length >= minApprovers) {
          return { approved: true, approvers: approvals };
        }
      }
    }
    
    return { approved: false, received: approvals.length };
  }
};
```

## Related modules and next step

- Previous: [08-workflows](../08-workflows/)
- Next: [10-cli-and-reference](../10-cli-and-reference/)
- Examples: [Approval patterns](approval-patterns.md)

---

**Time to complete:** 45 minutes  
**Prerequisites:** [08-workflows](../08-workflows/)  
**Outcome:** Secure, auditable, production-ready OpenClaw deployment
