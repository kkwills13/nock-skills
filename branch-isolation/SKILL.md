---
name: "branch-isolation"
description: "Branch discipline enforcement for Claude Code agents running in parallel. Use when multiple agents or sessions are working on the same repo simultaneously. Prevents an agent from accidentally switching to another agent's branch and contaminating diffs."
---

# Branch Isolation

When multiple Claude Code agents work on the same repo in parallel, branch discipline is critical. Without it, one agent can accidentally check out another's branch, contaminate their diff, and break CI for both.

This skill documents the patterns and tools for keeping agents in their lane.

## The problem

Claude Code is capable but not infallible. Given a complex enough prompt, it may:
- `git checkout` a branch to inspect something
- `git merge` main to resolve a conflict
- `git rebase` a branch mid-task

If Agent A switches to Agent B's branch, it can:
- Commit work to the wrong branch
- Create merge conflicts that weren't there before
- Make Agent B's PR show unrelated changes in the diff

## Solution 1: Branch-lock hook (automated)

A `PreToolUse` hook that blocks branch-switching commands when a session lock is active.

**How it works:**
1. On the first branch-sensitive git operation of a session, the hook writes the current branch to `.branch-lock` in the repo root.
2. On every Bash tool call, the hook checks if the command would switch to a different branch.
3. If it would, the hook exits 1 and Claude sees: `BRANCH LOCK: session is locked to 'X'. Blocked switch to 'Y'.`

**Install:**

Create `.claude/hooks/branch-lock.sh` (see [nock-skills/branch-isolation](https://github.com/nocktechnologies/nock-skills)):

```bash
# Register in .claude/settings.json:
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "bash \"$CLAUDE_PROJECT_DIR\"/.claude/hooks/branch-lock.sh",
        "timeout": 5
      }]
    }]
  }
}
```

**Allowed operations:** `git pull`, `git fetch`, `git add`, `git commit`, `git push`, `git checkout -- file`, `git checkout -b new-branch`, `git switch -c new-branch`

**Blocked operations:** `git checkout other-branch`, `git switch other-branch`, `git merge other-branch`, `git rebase other-branch`

**Reset:** `rm .branch-lock`

Add `.branch-lock` to `.gitignore`.

## Solution 2: Worktree isolation (stronger)

Git worktrees give each agent a completely separate working directory pointing to a different branch. No shared working tree = no accidental checkout.

```bash
# Agent A's setup
git worktree add /tmp/agent-a-workspace feature/my-feature

# Agent B's setup  
git worktree add /tmp/agent-b-workspace feature/other-feature
```

Each agent operates in its own directory. They can't see each other's unstaged changes. Switching branches in one worktree doesn't affect the other.

**Use worktrees when:** agents are running simultaneously in the same terminal session, or when you can't use hooks.

## Solution 3: Explicit CLAUDE.md rules

The simplest option: state the rule explicitly in your project's `CLAUDE.md`:

```markdown
## Branch discipline

You are working on branch `[branch-name]`.

Do NOT run:
- git checkout [any other branch]
- git switch [any other branch]
- git merge [any other branch]
- git rebase [any other branch]

If you need to inspect another branch, read its files via `git show branch:file` instead.
```

This works for well-behaved agents following instructions, but doesn't prevent mechanical errors.

## Combining approaches

For the highest reliability, combine all three:
1. **Worktrees**: separate working directories at the OS level
2. **Branch-lock hook**: automated enforcement inside the session
3. **CLAUDE.md rule**: explicit instruction that's always in context

## Signs you need this

- PRs showing unrelated changes in the diff
- CI failing on a branch because it contains commits from another branch  
- Merge conflicts appearing in files the agent wasn't supposed to touch
- "I didn't change that file" appearing in a PR review
