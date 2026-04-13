---
name: "session-handoffs"
description: "Structured context transfer between Claude Code sessions. Use at session end to write a handoff, and at session start to read one. Prevents context loss when working across multiple sessions on a long-running task."
---

# Session Handoffs

A simple, structured way to transfer context between Claude Code sessions. When you're doing multi-session work, a handoff is the difference between picking up exactly where you left off and spending 20 minutes re-reading code to figure out what you were doing.

## When to use

- Any task that spans more than one Claude Code session
- Before a hard restart or context reset
- When handing work off to another agent or model
- When you want a clear record of what's in-progress

## Writing a handoff (session end)

At the end of your session, write a handoff document. Keep it under 400 words. Include:

```markdown
# Handoff — [date]

## What I was working on
[One sentence describing the task]

## Current state
[What's done, what's in-progress, what's next]

## Critical context
[Anything a fresh session would miss: non-obvious decisions, gotchas, important constraints]

## Action items
- [ ] [Next concrete step]
- [ ] [Step after that]
- [ ] [Any blockers or decisions needed]

## Files in play
[List of files most relevant to the current work]

## Do NOT do
[Explicit warnings: what to avoid, what's already been tried and failed]
```

**Save as:** `HANDOFF.md` in the project root, or write it to your team's shared handoff system.

## Reading a handoff (session start)

At the start of a new session:

1. Read the handoff document first, before any code.
2. Verify the claimed state is accurate — check the files mentioned.
3. If action items are checked off, update the handoff before proceeding.
4. Identify any "Do NOT do" items and load them into working memory.

## What makes a good handoff

**Good:**
- "The auth middleware refactor is ~70% done. `core/auth.py` is updated. `views.py` still uses the old pattern on lines 44–67. Don't touch `tests/test_auth.py` — I wrote new tests but they're commented out until the refactor is done."
- "Decided to use a DB-level lock (SELECT FOR UPDATE) instead of Redis. See the ADR in `.claude/decisions/`."

**Bad:**
- "Working on the auth stuff."
- "In progress."
- Summaries so long they're faster to re-derive from the code

## Handoff for multi-agent work

When multiple agents are working in parallel:

1. Each agent maintains its own handoff for its branch.
2. Handoffs should note which branches/files are off-limits to other agents.
3. Include the expected PR number or branch name so agents can avoid collision.

Example:
```markdown
## Branch isolation note
I'm working on `feature/auth-refactor`. 
Codex is on `feature/data-model-v2`.
Do NOT touch: `models/user.py`, `migrations/` (Codex owns these)
```

## Minimal handoff template

For short tasks, use this stripped-down version:

```markdown
## Where I left off
[One sentence]

## Next step
[One sentence]

## Watch out for
[One sentence]
```
