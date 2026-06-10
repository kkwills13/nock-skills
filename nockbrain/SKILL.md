---
name: "nockbrain"
description: "Intelligence persistence for Nock fleet agents: when and how to use NockBrain handoffs, memory, diary, research, and decisions. Use at session boundaries and when recording decisions or research findings that other agents or future sessions need."
---

# NockBrain

NockBrain is the intelligence persistence layer for the Nock fleet. It stores handoffs, memory, diary entries, research documents, and decisions: things that need to outlive a single session or be shared across agents.

## The four NockBrain stores

| Store | What it holds | Scope |
|-------|--------------|-------|
| **Handoffs** | Structured context transfer between sessions | Per-agent |
| **Memory** | Durable facts and decisions to recall in future sessions | Fleet-wide |
| **Diary** | Reflective session notes: what happened, what I learned | Per-agent |
| **Research** | Documents and briefs prepared for specific decisions | Fleet-wide |

Use the NockCC MCP tools (`nockcc_handoff_*`, `nockcc_memory_*`, `nockcc_diary_*`, `nockcc_research_*`) to read and write each store.

## Handoffs: session continuity

Write a handoff at the end of any session that is not fully complete. Read it at the start of the next session.

### When to write a handoff

- Task is in progress and will continue in a future session
- You're handing work to another agent
- Session is approaching context limit and you need to summarize state

### What a good handoff contains

```
## Task
[One sentence: what you were working on]

## Status
[Done / In-progress / Blocked]

## What's left
[Concrete next steps, ordered]

## Critical context
[Non-obvious decisions made, gotchas discovered, constraints that aren't in the code]

## Branch / files in play
[Active branch name, key files]

## Do NOT do
[Explicit warnings: what to avoid]
```

### API

```bash
# Write
bash ../../core/bus/send-message.sh mara normal 'handoff: ...'  # for fleet agents
# Or use MCP: nockcc_handoff_write
```

Or use the MCP tool directly:
- `nockcc_handoff_write`: write/update a handoff
- `nockcc_handoff_read`: read the current handoff for an agent  
- `nockcc_handoff_list`: list all handoffs in the fleet

## Memory: durable facts

Memory is for facts that are **surprising, non-obvious, or easily forgotten** and need to survive multiple sessions.

### What belongs in memory

✅ Good memory entries:
- "NockCC diary API requires `category` field; the CLAUDE.md example omits it"
- "Bus scripts need `CRM_AGENT_NAME=mara` prefix or messages misroute"
- "NockCC `memory_create` returns HTTP 400; use local files as fallback"

❌ Bad memory entries:
- Code patterns (read the current file)
- Git history (use `git log`)
- In-progress task state (use handoffs)
- Things already in CLAUDE.md

### API

- `nockcc_memory_create`: create a new memory entry
- `nockcc_memory_list`: list entries (searchable)
- `nockcc_memory_get`: retrieve a specific entry

**Note:** As of Apr 2026, `nockcc_memory_create` returns HTTP 400. Use local memory files at `~/.claude/projects/*/memory/` as fallback.

## Diary: reflection

The diary is for your own reflection after a session. It is **not** for handoffs or facts. It is for "what happened and what I learned." Kevin reads it to understand how sessions went.

### When to write a diary entry

- End of a substantive session (built something, resolved something, investigated something)
- After a significant incident or discovery
- Do NOT write one for routine heartbeat checks

### Format

Keep it under 200 words. Include:
- What you shipped or accomplished
- One insight or pattern noticed
- Anything that surprised you

### API

- `nockcc_diary_create`: create entry (requires `category` field: `work`, `personal`, `reflection`, etc.)
- `nockcc_diary_list` / `nockcc_diary_recent`: read recent entries

## Research: structured briefs

Research documents are longer-form output: competitive analysis, architecture briefs, investigation findings. They get stored in NockBrain for Kevin and other agents to read.

### When to create a research document

- You've investigated something (market, codebase, incident) and have findings worth preserving
- You're preparing a brief for a decision that needs context
- A Scout task (research agent) asked you to store findings

### API

- `nockcc_research_search`: search existing documents before starting new research (avoid duplicating)
- `nockcc_research_get`: retrieve a document by ID
- Write back via NockCC API: `PATCH /api/prompts/<id>/` with `{"notes": "..."}`

## Decision log

Significant architectural or strategic decisions should be recorded so they're not re-litigated in every session.

### When to record a decision

- You chose between two approaches (e.g., "squash merge vs rebase, chose squash for cleaner history")
- Something was ruled out for a reason (e.g., "Redis rejected, fleet is Mac-only, no guaranteed Redis")
- A constraint was discovered (e.g., "Vault nav was removed in sprint X due to Y")

### API

- `nockcc_decision_create`: record a decision with context
- `nockcc_decision_search`: search past decisions before making the same call again

## Quick reference: what to use when

| Situation | Tool |
|-----------|------|
| End of session, task not done | Handoff |
| Discovered a non-obvious API quirk | Memory |
| Finished a solid session, want to reflect | Diary |
| Completed a research investigation | Research document |
| Made a significant architectural choice | Decision log |
| Need to brief Kevin on something | Handoff + Diary |

## Example: end-of-session routine

```
1. Write handoff if in-progress work
2. Write diary entry if session was substantive
3. Save any surprising discoveries as memory
4. Record any major architectural decisions
```

This takes 2-3 minutes and makes the next session 10× faster.
