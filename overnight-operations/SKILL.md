---
name: "overnight-operations"
description: "Patterns for AI agents that run while the human sleeps. Heartbeat cadence, dispatch authority, merge rules, notification thresholds, and sleep-mode criteria. Use when configuring an agent for overnight or unattended operation."
---

# Overnight Operations

Running AI agents while nobody's watching requires different patterns than interactive sessions. This skill covers the operational discipline for overnight and unattended agent operation.

## Core principle

**Dispatch or sleep.** If there's dispatchable work, dispatch it. If there's nothing to dispatch, sleep. Never burn compute on idle heartbeats with nothing to show for them.

## Heartbeat pattern

A heartbeat is a periodic check-in that polls inboxes, verifies system health, and takes action if needed.

### What a heartbeat checks

1. **Message inboxes**: new messages from other agents or the user
2. **PR pipeline**: any PRs ready for review or merge
3. **System health**: are services up, are agents responsive
4. **Task queue**: any queued work that can be dispatched

### Heartbeat cadence

- **Active overnight (dispatching work):** 5-15 minutes
- **Quiet overnight (monitoring only):** 15-30 minutes
- **Sleep mode (nothing happening):** stop heartbeating, rely on external triggers

### Heartbeat output

Every heartbeat should produce either:
- **Action taken**: "Merged PR #445, dispatched agent on N977"
- **HEARTBEAT_OK**: Nothing needed, all clear

If three consecutive heartbeats produce HEARTBEAT_OK with no pending work, consider entering sleep mode.

## Dispatch authority overnight

### DO dispatch

- Queued Nocks/tasks assigned to available agents
- Follow-up work from completed PRs (review, merge, next task)
- Automated responses to agent questions

### DO NOT dispatch

- New feature work not in the queue
- Work requiring user decisions or approval
- Anything that changes public-facing products (websites, published packages)

### Merge authority

- Merge PRs that pass all CI checks and have been reviewed
- Do NOT merge if CI is red, even partially
- Do NOT merge to public repos (open source, production sites) without explicit user authorization
- If unsure, queue it for the user's morning review

## Notification thresholds

### Wake the user for

- Fleet-wide service outage
- Security incident
- Financial/billing alert
- Something broke that can't wait until morning

### DO NOT wake the user for

- Individual PR merges
- Routine heartbeat results
- Non-blocking agent issues
- Questions that can wait 6 hours

### Morning brief

At a configured time (e.g., 6 AM), compile everything that happened overnight into a single message:

```
Morning brief: May 20, 2026

Shipped:
- PR #445 auth middleware (merged 2:15 AM)
- PR #446 quota limits (merged 3:40 AM)

In progress:
- Agent working on N977 since 4 AM

Needs your attention:
- PR #447 has a CI failure I can't diagnose
- Agent asked a question about the pricing model

Spend: $X.XX overnight
```

## Sleep mode

When there's no dispatchable work and inboxes are empty:

1. Write a checkpoint/handoff with current state
2. Set a long-interval check (30-60 minutes) or stop heartbeating
3. Resume active mode when a message arrives or the user wakes up

The goal: don't burn tokens watching nothing happen.

## Compaction survival overnight

Long-running overnight sessions will hit context limits. Use the [compaction-survival](../compaction-survival/) skill:

- Write checkpoints every 15 minutes
- Write external checkpoint files so state survives session restarts
- Include the dispatch table and open PR state in every checkpoint

## Example overnight config

```json
{
  "crons": [
    {
      "name": "heartbeat",
      "interval": "15m",
      "prompt": "Check inboxes, review PR pipeline, dispatch queued work. HEARTBEAT_OK if nothing needed."
    },
    {
      "name": "checkpoint",
      "interval": "15m",
      "prompt": "Write state checkpoint to file."
    },
    {
      "name": "morning-brief",
      "interval": "6am",
      "prompt": "Compile overnight activity into a morning brief."
    }
  ]
}
```

## Anti-patterns

- **Idle heartbeat loops**: Running every 5 minutes for 8 hours with nothing to do. Waste of compute.
- **Unilateral decisions**: Making product or strategic decisions at 3 AM without the user. Queue them.
- **Notification spam**: Sending the user 15 messages about individual PRs. Bundle into the morning brief.
- **Silent failures**: A service goes down at 1 AM and nobody notices until 9 AM. If something breaks, notify immediately.
- **Scope creep**: "While I'm waiting, let me refactor this unrelated thing." Don't. Stick to the queue.
