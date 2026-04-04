# Task Flow: Background Orchestration

> Durable background task orchestration with managed/mirrored sync modes, revision tracking, and graceful recovery.

**Version introduced**: v2026.4.2  
**Level**: Intermediate to Advanced  
**Prerequisites**: [06-automation: Cron Jobs](./cron-jobs.md), [11-creating-agents](../11-creating-agents/)

---

## What is Task Flow?

Task Flow is OpenClaw's foundational layer for orchestrating long-running background tasks. Unlike simple cron jobs that fire and forget, Task Flow provides:

- **Durable state** that survives gateway restarts
- **Revision tracking** for precise recovery points
- **Child task spawning** with coordinated lifecycle management
- **Sticky cancel intent** for graceful shutdowns
- **Plugin API access** for building orchestration capabilities

Use Task Flow when you need:
- Multi-step workflows that must survive interruptions
- Parent-child task relationships with coordinated cancellation
- Background pipelines (data extraction → processing → reporting)
- Complex automation that requires state inspection and recovery

---

## Sync Modes

Task Flow supports two synchronization modes:

### Managed Mode (Default)

OpenClaw fully manages and persists the flow state. The flow's lifecycle is decoupled from any external orchestration system.

**Use when**: You want OpenClaw to be the source of truth for task orchestration.

```json5
{
  taskFlows: {
    myPipeline: {
      mode: "managed",
      steps: [
        { name: "extract", tool: "web_fetch" },
        { name: "process", agent: "data-processor" },
        { name: "report", channel: "slack" }
      ]
    }
  }
}
```

### Mirrored Mode

The flow state mirrors an external orchestration system. OpenClaw aligns its state with external state changes.

**Use when**: You have an external orchestrator (Airflow, Temporal, etc.) and OpenClaw executes tasks within that framework.

```json5
{
  taskFlows: {
    externalPipeline: {
      mode: "mirrored",
      externalId: "airflow-dag-123",
      syncInterval: "30s"
    }
  }
}
```

---

## Basic Task Flow

### Creating a Flow

```bash
# Create a simple 3-step workflow
openclaw taskflow create \
  --name daily-report \
  --steps "fetch-data,process,notify"
```

### Inspecting Flows

```bash
# List all active flows
openclaw flows list

# Get detailed state of a specific flow
openclaw flows status daily-report

# View revision history
openclaw flows revisions daily-report
```

### Triggering and Cancelling

```bash
# Start a flow
openclaw flows start daily-report

# Cancel with sticky intent (waits for child tasks)
openclaw flows cancel daily-report --graceful

# Force cancel (immediate termination)
openclaw flows cancel daily-report --force
```

---

## Child Task Spawning

Task Flow supports parent-child relationships where child tasks inherit context but have independent lifecycle management.

### Parent Flow with Children

```json5
{
  taskFlows: {
    contentPipeline: {
      mode: "managed",
      steps: [
        {
          name: "research",
          spawn: {
            agent: "researcher",
            isolated: true,
            timeout: "10m"
          }
        },
        {
          name: "write",
          dependsOn: ["research"],
          spawn: {
            agent: "writer",
            context: "{{steps.research.output}}"
          }
        },
        {
          name: "thumbnail",
          parallel: true,  // Run alongside "write"
          spawn: {
            agent: "designer",
            task: "generate thumbnail"
          }
        }
      ]
    }
  }
}
```

### Sticky Cancel Intent

When you cancel a parent flow, child tasks receive cancel intent but can complete gracefully:

```bash
# Parent signals cancel, children finish current work then stop
openclaw flows cancel contentPipeline --graceful

# Check child task status
openclaw flows children contentPipeline
```

---

## Recovery and Resilience

### Automatic Recovery

If the gateway restarts, Task Flow automatically recovers to the last committed revision:

```bash
# Check recovery status after restart
openclaw flows status daily-report

# Expected output:
# Flow: daily-report
# State: recovered
# Revision: 42
# LastStep: process
# ChildTasks: 2 active
```

### Manual Recovery

If automatic recovery fails or you need to resume from a specific point:

```bash
# List available recovery points
openclaw flows revisions daily-report --failed

# Resume from specific revision
openclaw flows resume daily-report --from-revision 38

# Retry failed step only
openclaw flows retry daily-report --step notify
```

---

## Plugin API Access

Plugins can create and manage Task Flows through the `api.runtime.taskFlow` seam:

```typescript
// Plugin code example
export async function execute(context) {
  const { taskFlow } = context.api.runtime;
  
  // Create a managed flow
  const flow = await taskFlow.create({
    name: "plugin-orchestrated",
    mode: "managed",
    steps: [
      { tool: "web_search", query: context.input.topic },
      { agent: "summarizer", input: "{{step0.results}}" }
    ]
  });
  
  // Start and monitor
  await flow.start();
  const status = await flow.waitForCompletion({ timeout: "5m" });
  
  return { status, output: status.output };
}
```

This allows plugins to orchestrate complex workflows without requiring external identifiers on each call.

---

## Task Flow vs Cron Jobs

| Feature | Cron Jobs | Task Flow |
|---------|-----------|-----------|
| **Trigger** | Time-based | Event, API, or time-based |
| **State** | Stateless | Durable with revision tracking |
| **Recovery** | None | Automatic recovery from restart |
| **Child tasks** | No | Managed child task spawning |
| **Complexity** | Simple one-offs | Multi-step orchestration |
| **Use case** | Briefings, simple polls | Pipelines, workflows, multi-agent |

---

## Real-World Example: Content Factory

A multi-agent content production pipeline:

```json5
{
  taskFlows: {
    newsletterFactory: {
      mode: "managed",
      schedule: "0 7 * * 1",  // Mondays at 7am
      steps: [
        {
          name: "curate",
          spawn: {
            agent: "researcher",
            task: "Find top 5 AI news from the past week"
          }
        },
        {
          name: "draft",
          dependsOn: ["curate"],
          spawn: {
            agent: "writer",
            task: "Write newsletter draft from {{steps.curate.articles}}"
          }
        },
        {
          name: "review",
          dependsOn: ["draft"],
          spawn: {
            agent: "editor",
            task: "Review and edit: {{steps.draft.content}}"
          }
        },
        {
          name: "publish",
          dependsOn: ["review"],
          channel: "email",
          to: "subscribers@example.com",
          body: "{{steps.review.finalContent}}"
        }
      ],
      onFailure: {
        notify: "admin@example.com",
        retry: { maxAttempts: 3, backoff: "exponential" }
      }
    }
  }
}
```

**Recovery scenario**: Gateway restarts during "draft" step → Flow resumes at "draft" with research context intact.

---

## CLI Reference

| Command | Description |
|---------|-------------|
| `openclaw flows list` | List all flows |
| `openclaw flows status <name>` | Get flow state and revision |
| `openclaw flows revisions <name>` | Show revision history |
| `openclaw flows start <name>` | Trigger a flow |
| `openclaw flows cancel <name>` | Cancel with graceful shutdown |
| `openclaw flows resume <name>` | Resume from last revision |
| `openclaw flows retry <name>` | Retry failed steps |
| `openclaw flows children <name>` | List child tasks |
| `openclaw flows logs <name>` | View execution logs |

---

## Troubleshooting

### "Flow state lost after restart"

**Check**: Ensure flow is using `mode: "managed"` or `mode: "mirrored"`, not legacy stateless mode.

**Fix**: Update config to explicit mode declaration.

### "Child tasks orphaned after cancel"

**Check**: Verify you're using `--graceful` flag for sticky cancel intent.

**Fix**: 
```bash
openclaw flows cancel <name> --graceful
# Wait for children, then verify:
openclaw flows children <name>
```

### "Cannot resume from revision"

**Check**: Revision may be pruned. Check retention settings.

**Fix**: Adjust retention or retry from available revision:
```bash
openclaw flows revisions <name> --available
openclaw flows resume <name> --from-revision <id>
```

---

## Next Steps

- [Combine with Cron](./cron-jobs.md) for scheduled orchestration
- [Build Multi-Agent Workflows](../11-creating-agents/multi-agent-patterns.md)
- [Create Custom Plugins](../11-creating-agents/plugin-development.md) using Task Flow API

---

**Last Updated**: April 4, 2026  
**Applies To**: OpenClaw v2026.4.2+
