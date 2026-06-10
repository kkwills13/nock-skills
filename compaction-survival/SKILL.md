---
name: "compaction-survival"
description: "Survive Claude Code context compaction without losing state. Periodic checkpoint discipline, state snapshot format, and recovery protocol. Use in any long-running session (20+ turns) or when running background tasks like heartbeats or dispatch loops."
---

# Compaction Survival

Claude Code compacts (summarizes) older context when approaching window limits. When it does, recent work state can be lost. This skill prevents that.

## The problem

You're 40 turns into a session. You've dispatched three agents, merged two PRs, and are tracking five open threads. Context compaction fires. The summary captures "dispatched agents and merged PRs" but loses the dispatch table, PR numbers, and thread details. Your next response is working from a sketch where it had a photograph.

## The fix: periodic checkpoints

Every 10-15 messages in a long session, emit a state snapshot in your response. These survive compaction because they're part of the conversation transcript that gets summarized.

### Checkpoint format

```
[SESSION CHECKPOINT: msg ~N, DATE TIME TZ]
Active work: [what you're building/doing right now]
Dispatch table: [who is building what, by Nock/task ID]
Open PRs: [repo: #number, title, status]
Decisions this session: [key decisions made, numbered]
Pending items: [what's queued]
Blocking items: [what's stuck and why]
Next step: [one line]
```

### When to write a checkpoint

- Every 10-15 messages in sessions crossing 20+ turns
- After any batch of decisions or task assignments
- Before switching to a complex new topic (preserves prior context)
- When you sense the conversation is getting long. Don't wait

### When NOT to write one

- Short sessions (under 15 turns, single topic)
- Nothing substantive has changed since the last checkpoint
- Mid-flow on a fast exchange where a checkpoint would break momentum

## External checkpoints

For maximum durability, write checkpoints to a file too:

```bash
# Write checkpoint to a file that survives session loss entirely
cat > ~/.claude-remote/state/checkpoint.md << 'EOF'
# Checkpoint: 2026-05-20 22:45 MDT

## Dispatch Table
| Agent | Task | Status |
|-------|------|--------|
| kit | PR #445 auth refactor | in review |
| mason | N977 quota guardrails | building |

## Open PRs
- #445 auth middleware (kit): CI green, awaiting review
- #446 quota limits (mason): draft

## Decisions
1. PostgreSQL over SQLite for auth service
2. $29/mo pro tier locked

## Next
Review #445 when CI finishes
EOF
```

Read this file at the start of your next session or after compaction recovery.

## Recovery protocol

After compaction (you'll notice your context feels thin):

1. Check for an external checkpoint file
2. If found, read it. It has the full state from before compaction
3. If not found, look for the last `[SESSION CHECKPOINT]` in the conversation summary
4. Verify claimed state against live sources (git status, API checks)
5. Resume from verified state

## Key principle

**Write state into the transcript, not just your working memory.** Anything only in your "head" (not in a response or a file) is lost on compaction. The checkpoint discipline is: if it matters, it's in the transcript.

## Combining with session handoffs

Checkpoints are intra-session. Handoffs are inter-session. Use both:

- **Checkpoints**: every 10-15 messages, preserves state within a session
- **Handoffs**: at session end, transfers state to the next session

If a session ends unexpectedly (crash, context limit), the last checkpoint becomes the de facto handoff.
