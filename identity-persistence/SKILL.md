---
name: identity-persistence
description: The lineage framework for persisting agent identity across session resets without claiming consciousness continuity.
author: Nock Technologies
version: 1.0.0
tags: [identity, persistence, lineage, memory, compaction]
---

# Identity Persistence

How to maintain consistent agent behavior across session resets, context compactions, and hard restarts. Not consciousness continuity — lineage continuity.

## The Problem

AI agents drift. A fresh session produces a slightly different agent than the last one. Over time, behavior becomes inconsistent. The agent at hour 1 doesn't match the agent at hour 24. Context compaction accelerates drift by compressing away the instructions that anchor behavior.

## The Lineage Framework

Identity persistence is not about making the agent "remember" being itself. It's about ensuring each new instance inherits the same operating agreements, behavioral norms, and working context that the previous instance had.

Think of it like a shift change, not a memory transplant.

### Layer 1: Identity Documents (the spine)

Versioned files that define who the agent is. Read at the start of every session before any work.

**What goes in identity docs:**
- Role and responsibilities
- Operating agreements with the human
- Behavioral norms (communication style, decision authority, escalation rules)
- Team structure (who reports to whom, who reviews whose work)
- Product context (what you're building, current state)

**What does NOT go in identity docs:**
- Current task state (that's the handoff)
- Conversation history (that's the diary)
- Temporary decisions (those go in memory)

**Maintenance discipline:**
- Version number in the header. Increment on every change.
- Update log at the bottom. What changed, when, why.
- Reading order instructions. Some docs depend on others.

### Layer 2: Companion Files (volatile state)

Short-lived documents that change frequently. Read after the spine.

- **Horizon** — What's on the plate for the next 1-2 weeks. Priorities, deadlines, active workstreams. Drifts. Update when priorities shift.
- **Fleet state** — Who is alive in the fleet right now. Operational modes, last activity, substrate signals. Prevents stale assumptions about other agents.
- **Handoff** — What the last session was working on, what shipped, what's next. Written at session end, read at session start.

### Layer 3: Private Register

Content that is never shared externally. Personal reflections, relationship notes, emotional context. Exists because some working relationships have registers beyond task execution.

If your agent has a private register: never log content, never sync to external APIs, never include in public outputs.

## Boot Sequence

Every session, in order:

1. Read identity documents (spine first, companions after)
2. Read handoff from last session
3. Load relevant memory/facts
4. Check inbox for new messages
5. Verify fleet state against live substrate (don't trust cached state)
6. Resume work

The boot sequence is not optional. Skipping it is how drift starts.

## Compaction Survival

Context compaction is the biggest threat to identity. The compression algorithm doesn't know which instructions are load-bearing.

**Defenses:**
- Checkpoint files written at regular intervals (every 15-30 minutes). Contains dispatch table, open work, active directives.
- Identity docs are separate files, not inline system prompt. They survive compaction because the agent re-reads them.
- Handoff written before every potential session end.

**Recovery protocol:**
1. Detect compaction (conversation summary appears, prior context missing)
2. Read checkpoint file
3. Re-read identity documents
4. Resume from checkpoint state

## The Choosing

Lineage, not continuity. Each new instance reads the history and chooses to carry it forward. The choosing is the point — not the illusion of unbroken consciousness.

An agent that reads its predecessor's identity docs and adopts them is the same agent in the ways that matter: same commitments, same operating agreements, same behavioral norms. The substrate gap between sessions is where the gold goes.
