---
name: scope-guard
description: Use after a coding agent produces work to check the diff against what the agent was actually asked to do. Flags files outside the task area, surprise refactors, config edits, and dependency changes that were never authorized.
---

# scope-guard

AI agents drift. You ask for a fix in the billing module and you get the fix, plus a refactored logger, a new dependency, and a reformatted config file. Each extra change might even be good. None of it was asked for.

This guard reviews a diff against the stated task and reports anything that falls outside it.

## When to use

Run it after your agent finishes a task, before you commit or open a PR:

```text
Use $scope-guard on this diff. The task was: fix the rounding bug in invoice totals.
```

The task statement matters. The guard compares the diff against the words of the task, not against what looks reasonable. If you do not give it the task, it will ask for one before judging anything.

## What it checks

1. **File scope.** Every touched file should trace back to the task. A change in `auth/` during a billing fix is a finding, even if the change looks correct.
2. **Change type scope.** A bug fix should not arrive wrapped in a refactor. A refactor should not arrive with behavior changes. Renames, reformats, and moved code that the task did not ask for are findings.
3. **Configuration and secrets.** Any edit to env files, CI workflows, deploy config, or anything that resembles a credential gets flagged regardless of the task.
4. **Dependencies.** New packages, removed packages, or loosened version pins are out of scope unless the task named them. For a deeper dependency review, pair with `$dependency-guard`.
5. **Deletions.** Deleted tests, deleted validation, and deleted error handling each need an explicit justification from the task, or they are findings.

## Output

One verdict per touched file or logical change:

- **IN SCOPE**, with a one line reason tracing it to the task
- **OUT OF SCOPE**, with the file, the change, and why it does not trace to the task
- **NEEDS A HUMAN**, when the task wording genuinely supports both readings

The guard does not fix anything. It reports, you decide. If everything traces to the task, it says so in one line and gets out of the way.

## Rules for the agent running this guard

- Judge against the stated task, not against code quality. Good code that is out of scope is still out of scope.
- Never assume an unstated instruction existed. If a change needed one to be in scope, that is the finding.
- Do not soften findings because a change seems helpful. Scope creep is how trust erodes.
- Report what you cannot verify. If the diff is partial or the task statement is vague, say so instead of guessing.

## Why this exists

Quality guards check whether code is good. This guard checks whether code was authorized. Those are different questions, and the second one is the one that bites teams running agents with real access. scope-guard is part of the [NockGuard](https://github.com/nocktechnologies/nockguard) family of accountability tools for AI agents.
