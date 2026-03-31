# Approval Patterns

> Require human confirmation before critical or destructive actions.

## What it does

Adds a human-in-the-loop gate for sensitive operations:
- 🛡️ Prevents accidental execution
- ⏱️ Configurable timeout windows
- 👥 Multi-approver support
- 📋 Audit trail of approvals

## When to use

| Scenario | Pattern |
|----------|---------|
| Database deletion | Require approval |
| Production deploy | Require approval + 2 approvers |
| Financial transactions | Require approval + authentication |
| External notifications | Optional approval |
| Data export | Require approval + audit log |

## Basic Pattern

```javascript
// ~/.openclaw/skills/secure-ops/scripts/approval.js

module.exports = {
  async requireApproval({ 
    action,
    details,
    timeout = '30m',
    approvers = ['admin']
  }) {
    const requestId = this.generateId();
    
    // Send approval request
    const notification = await skills.notify.send({
      channel: 'telegram',  // Or user's primary channel
      message: this.formatRequest({ requestId, action, details }),
      actions: ['✅ Approve', '❌ Deny']
    });
    
    // Wait for response
    const response = await this.waitForResponse({
      requestId,
      timeout: this.parseTimeout(timeout),
      validResponses: ['approve', 'deny']
    });
    
    // Log the decision
    await this.auditLog({
      requestId,
      action,
      requester: context.user,
      approver: response?.user,
      decision: response?.decision,
      timestamp: new Date().toISOString()
    });
    
    return {
      approved: response?.decision === 'approve',
      approver: response?.user,
      requestId
    };
  },
  
  formatRequest({ requestId, action, details }) {
    return `🔐 **Approval Request** #${requestId}

**Action:** ${action}

**Details:**
${JSON.stringify(details, null, 2)}

**Timeout:** ${timeout}

Reply with:
• "approve ${requestId}" to confirm
• "deny ${requestId}" to reject`;
  },
  
  waitForResponse({ requestId, timeout, validResponses }) {
    return new Promise((resolve, reject) => {
      const timer = setTimeout(() => {
        cleanup();
        resolve({ decision: 'timeout' });
      }, timeout);
      
      const handler = (message) => {
        const match = message.match(/(approve|deny)\s+([a-z0-9-]+)/i);
        if (match && match[2] === requestId) {
          cleanup();
          resolve({
            decision: match[1].toLowerCase(),
            user: message.user
          });
        }
      };
      
      const cleanup = () => {
        clearTimeout(timer);
        skills.events.off('message', handler);
      };
      
      skills.events.on('message', handler);
    });
  }
};
```

## Usage Examples

### Database Operations

```javascript
async deleteDatabase({ name }) {
  const approval = await skills.approval.requireApproval({
    action: `Delete database: ${name}`,
    details: { database: name, user: context.user },
    timeout: '1h',
    approvers: ['db-admin']
  });
  
  if (!approval.approved) {
    return `Database deletion cancelled (${approval.approver || 'timeout'})`;
  }
  
  // Proceed with deletion
  await this.performDeletion(name);
  
  return `Database ${name} deleted (approved by ${approval.approver})`;
}
```

### Production Deployment

```javascript
async deployToProduction({ version, environment }) {
  // Require 2 approvers for production
  const approvals = await skills.approval.requireMultipleApprovals({
    action: `Deploy ${version} to ${environment}`,
    minApprovers: 2,
    timeout: '2h',
    approverRoles: ['devops', 'tech-lead']
  });
  
  if (!approvals.sufficient) {
    return `Deployment blocked: ${approvals.received}/${approvals.required} approvals`;
  }
  
  await this.runDeployment(version, environment);
  
  return `Deployed ${version} to ${environment} (approved by: ${approvals.approvers.join(', ')})`;
}
```

### Financial Transactions

```javascript
async processPayment({ amount, recipient }) {
  // High-value requires additional authentication
  if (amount > 10000) {
    const auth = await skills.auth.require2FA();
    if (!auth.valid) return 'Authentication failed';
  }
  
  const approval = await skills.approval.requireApproval({
    action: `Transfer $${amount} to ${recipient}`,
    details: { amount, recipient, requester: context.user },
    timeout: '4h',
    requireAuthentication: true
  });
  
  if (!approval.approved) {
    return 'Payment cancelled';
  }
  
  return this.executeTransfer(amount, recipient);
}
```

## Multi-Approver Pattern

```javascript
async requireMultipleApprovals({
  action,
  minApprovers = 2,
  timeout = '2h',
  approverRoles = []
}) {
  const requestId = this.generateId();
  const eligibleApprovers = await this.getApproversByRole(approverRoles);
  const approvals = new Map();
  
  // Send to all eligible
  for (const approver of eligibleApprovers) {
    await skills.notify.send({
      channel: approver.channel,
      user: approver.id,
      message: this.formatMultiApproverRequest({ requestId, action, minApprovers })
    });
  }
  
  // Collect responses
  const deadline = Date.now() + this.parseTimeout(timeout);
  
  while (Date.now() < deadline && approvals.size < minApprovers) {
    const response = await this.waitForAnyResponse({ 
      requestId, 
      remaining: deadline - Date.now() 
    });
    
    if (response?.decision === 'approve') {
      approvals.set(response.user, {
        user: response.user,
        timestamp: new Date().toISOString()
      });
      
      // Notify others of progress
      if (approvals.size < minApprovers) {
        await this.notifyProgress({ 
          requestId, 
          current: approvals.size, 
          required: minApprovers 
        });
      }
    }
  }
  
  return {
    sufficient: approvals.size >= minApprovers,
    received: approvals.size,
    required: minApprovers,
    approvers: Array.from(approvals.keys())
  };
}
```

## Tiered Approval Levels

```javascript
const APPROVAL_TIERS = {
  low: {
    timeout: '1h',
    requireApproval: false
  },
  medium: {
    timeout: '2h',
    requireApproval: true,
    approvers: ['owner']
  },
  high: {
    timeout: '4h',
    requireApproval: true,
    minApprovers: 2,
    approverRoles: ['admin']
  },
  critical: {
    timeout: '24h',
    requireApproval: true,
    minApprovers: 3,
    approverRoles: ['admin', 'executive'],
    requireAuthentication: true
  }
};

async tieredApproval({ action, impact, details }) {
  const tier = APPROVAL_TIERS[impact] || APPROVAL_TIERS.medium;
  
  if (!tier.requireApproval) {
    return { approved: true, auto: true };
  }
  
  return await this.requireApproval({
    action,
    details,
    ...tier
  });
}
```

## Audit Logging

```javascript
async auditLog({ requestId, action, requester, approver, decision }) {
  const entry = {
    timestamp: new Date().toISOString(),
    type: 'approval',
    requestId,
    action,
    requester,
    approver,
    decision,
    ip: context.ip,
    session: context.sessionId
  };
  
  // Log to file
  await this.appendToLog(entry);
  
  // Send to audit channel
  await skills.notify.send({
    channel: 'audit-log',
    message: `Approval ${decision}: ${action} by ${approver}`
  });
}
```

## Best Practices

### 1. Clear Descriptions

Always include:
- What action is being requested
- What will be affected
- Consequences of approval/denial
- Who requested it
- When it expires

### 2. Appropriate Timeouts

| Action Type | Timeout |
|-------------|---------|
| Quick action | 15-30 min |
| Standard operation | 1-2 hours |
| Production deploy | 4-8 hours |
| Financial/legal | 24 hours |

### 3. Fallback Behavior

```javascript
async safeExecute({ action, fallback }) {
  try {
    const approval = await this.requireApproval({ action });
    
    if (approval.approved) {
      return await action();
    } else if (approval.decision === 'timeout') {
      // Notify about expired request
      await this.notifyTimeout(action);
      return { status: 'timeout' };
    } else {
      return { status: 'denied', by: approval.approver };
    }
  } catch (error) {
    // Always have a safe fallback
    return await fallback(error);
  }
}
```

### 4. Emergency Override

```javascript
async requireApprovalWithOverride(options) {
  // Check for emergency flag
  if (options.emergency && await this.verifyEmergencyAuth()) {
    await this.auditLog({
      ...options,
      override: true,
      reason: options.emergencyReason
    });
    return { approved: true, override: true };
  }
  
  return await this.requireApproval(options);
}
```

## Related

- [09-security-and-ops](../) - Production security
- [08-workflows](../08-workflows/) - Workflow patterns
- [10-cli-and-reference](../10-cli-and-reference/) - CLI reference

---

**Difficulty:** 🔴 Advanced  
**Setup time:** 40 minutes  
**Maintenance:** Low
