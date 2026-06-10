---
name: "fleet-heartbeat"
description: "Multi-channel polling pattern for AI agent fleets. Check N inboxes on a cadence, take action on messages, produce state snapshots for compaction survival. Use when running a coordinator agent that monitors multiple communication channels."
---

# Fleet Heartbeat

A structured pattern for agents that need to poll multiple channels, act on messages, and maintain operational awareness across a fleet of agents.

## The pattern

Every heartbeat cycle:

```
1. POLL    -> Check all channels for new messages
2. ACT     -> Process messages, dispatch work, merge PRs
3. SNAPSHOT -> Write state for compaction survival
4. SIGNAL  -> Notify the user only if action is needed
```

## Channel polling

Poll channels in priority order. Each channel check should be independent: a failure in one doesn't block the others.

```
Channel 1: API inbox (highest priority, cross-instance messages)
Channel 2: File bus (same-machine agent messages)
Channel 3: Chat/messaging (Telegram, Slack, etc.)
Channel 4: PR pipeline (GitHub, GitLab)
Channel 5: System health (services, agents, crons)
```

### Message handling

For each message received:
1. **Read** the message
2. **ACK** it (mark as read, reply with acknowledgment)
3. **Act** on it (dispatch work, answer question, merge PR)
4. **Log** what you did

Always include the message ID in your ACK so the sender knows which message was processed. Un-ACK'd messages may redeliver.

## State snapshot

Every heartbeat should output a concise state block. This serves two purposes:
- Survives context compaction (it's in the conversation transcript)
- Gives the user a quick status check

```
[STATE SNAPSHOT: 2026-05-20 03:15 MDT]
Dispatch: kit→N977 (building), mason→PR#445 (reviewing)
Open PRs: NCC #445 (CI green), #446 (draft)
Inboxes: 0 unread
Health: all services up
Blocking: nothing
```

Keep snapshots under 200 tokens. They're breadcrumbs, not reports.

## Signal discipline

After each heartbeat:
- **Action taken** → Log what you did, notify user only if significant
- **HEARTBEAT_OK** → One line, no notification needed
- **Problem detected** → Notify user immediately with: what, when, impact, fallback

## Cadence

| Mode | Interval | When |
|------|----------|------|
| Active dispatch | 5 min | Agents building, PRs in flight |
| Monitoring | 15 min | Quiet period, nothing pending |
| Sleep | 30+ min | No work, all inboxes empty |

Adapt cadence to activity. Three consecutive HEARTBEAT_OK in active mode → switch to monitoring.

## Multi-agent coordination

When coordinating multiple agents:

### Dispatch refueling

On each heartbeat, check for idle agents with queued work:
1. List agents and their current task (or idle status)
2. List queued tasks that match idle agents' capabilities
3. Dispatch: send task to agent via message bus
4. Record dispatch in state snapshot

### Agent health

Don't assert agent state from heuristics (log size, file timestamps). Either:
- Check a definitive signal (process list, API status endpoint)
- Say "unsure" rather than guessing

Wrong health assertions erode trust. "I don't know if the agent is stuck" is better than "the agent is stuck" when you're guessing.

## Error handling

- **Channel down:** Skip it, check others, note the failure in the snapshot
- **Agent unresponsive:** Don't restart mid-investigation. Note it, retry next heartbeat, escalate if persistent
- **Service outage:** Notify user immediately. Don't retry in a loop; diagnose first

## Example implementation

```bash
# Heartbeat cron prompt
"Pipeline check:
(1) Check API inbox for unread messages. Process and ACK.
(2) Check file bus for agent messages. Process and ACK.
(3) Poll chat for user messages. Act on them.
(4) Check PR pipeline for mergeable PRs. Review and merge if green.
(5) STATE SNAPSHOT: dispatch table, open PRs, blocking items."
```
