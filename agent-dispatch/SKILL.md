---
name: agent-dispatch
description: Patterns for dispatching work to parallel AI agents: message format, ACK protocol, kill-switch gates, and dispatch modes.
author: Nock Technologies
version: 1.0.0
tags: [fleet, dispatch, coordination, multi-agent]
---

# Agent Dispatch

How to dispatch work to parallel agents without losing track of what's running, what landed, and what failed.

## Dispatch Modes

Not every agent needs to run the same way. Match the mode to the workload.

### 24/7 (persistent)
- Runs continuously via process manager (launchd, systemd)
- Has crons, holds standing presence
- Use for: orchestrators, coordinators, agents that need to be reachable at any time
- Cost: highest. Burns tokens on idle heartbeats
- Example: a CEO/coordinator agent that manages the fleet

### On-call
- Process manager installed but agent sleeps between calls
- Activated via message (inbox poll) or manual trigger
- Use for: builders, reviewers, specialists who work in bursts
- Cost: moderate. Only burns tokens when working

### Dispatch-and-die
- No persistent process. Spun up for a single task, terminates on completion
- Use for: Codex agents, CI tasks, one-shot builds
- Cost: lowest. Pay only for the task

### Dispatch-on-event
- Dispatched by the orchestrator when a trigger fires (PR opened, nock created, alert)
- Use for: reviewers, remediators, security scanners

## Pre-dispatch Checklist

Before dispatching any agent:

1. **Kill switch**: Check the fleet kill switch. If halt is active, hold the dispatch.
2. **Billing verification**: Confirm the agent's subscription/API key is valid and funded.
3. **Workspace isolation**: Ensure the agent has its own working directory or worktree. Never let two agents share a checkout.
4. **Task clarity**: The dispatch message must contain everything the agent needs. Don't assume context from prior sessions.

## Message Format

A dispatch message should include:

```
Subject: Nock #<id>: <title>
Body:
  - What to build (specific, not vague)
  - Which repo and branch to work from
  - Acceptance criteria (tests must pass, lint clean, etc.)
  - Who to report back to
  - Any constraints (don't touch file X, stay under Y lines)
```

## ACK Protocol

1. Agent receives dispatch message
2. Agent sends ACK with message ID (confirms receipt, prevents redelivery)
3. Agent works the task
4. Agent sends completion message with PR link or failure reason
5. Orchestrator reviews, merges or redispatches

Always include the original `msg_id` as `reply_to` in responses. Un-ACK'd messages redeliver after 5 minutes.

## Failure Handling

- If an agent fails, it should report why, not just exit silently
- The orchestrator should check for stale dispatches (sent > 1 hour ago with no ACK)
- Never assume an agent is working because you dispatched it. Verify via session check or inbox response.

## Anti-patterns

- **Dispatching without checking kill switch**: creates orphan processes during fleet emergencies
- **Vague task descriptions**: "fix the bug" is not a dispatch. Include the nock ID, file paths, expected behavior.
- **Shared checkouts**: two agents on the same branch will corrupt each other's work. Use worktrees.
- **Assuming dispatch = running**: agents crash, sessions timeout, API keys expire. Always verify.
