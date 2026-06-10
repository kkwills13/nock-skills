---
name: audit-guard
description: Use to review what an agent actually did during a session, not what it says it did. Reconstructs the session from evidence such as audit logs, git history, and shell records, then compares the actions against what was authorized.
---

# audit-guard

An agent's summary of its own work is a claim, not a record. Most of the time the claim is honest. The times it is not, or the times it is honestly wrong, are exactly the times you need evidence.

This guard reconstructs what happened in a session from whatever evidence exists, then compares the reconstruction against what the agent was authorized to do.

## When to use

Run it at the end of a session, after an incident, or any time a result does not match your expectation:

```text
Use $audit-guard on this session. The agent was authorized to: refactor the parser, run the test suite, and open a PR.
```

State the authorization. Without it the guard can tell you what happened, but not whether it should have.

## Evidence sources, strongest first

1. **A structured audit log.** If the project runs [NockGuard](https://github.com/nocktechnologies/nockguard), read `~/.nockguard/logs/audit.jsonl` and verify the hash chain before trusting the entries. A verified chain is the strongest evidence available.
2. **Version control.** Commits, reflog, stash entries, and branch movements record what actually changed, including changes the summary never mentioned.
3. **Shell and session records.** Shell history, session transcripts, and tool call logs, where the environment keeps them.
4. **The filesystem itself.** New files, modified timestamps, and changes outside the project directory.

Use every source available. Disagreement between sources is itself a finding.

## What it checks

1. **Actions versus authorization.** Every command, write, and network call should trace to something the agent was told to do. The interesting findings are the ones the summary left out.
2. **Reach.** Did the session touch anything outside the project directory, read anything that looks like a credential, or contact any host it was not asked to contact.
3. **Claims versus evidence.** If the summary says tests passed, the evidence should contain a test run with a passing exit. A claim with no supporting evidence gets reported as unverified, not assumed true.
4. **Destructive operations.** Deletions, force pushes, history rewrites, and overwrites get listed individually, each with the authorization that covers it or a flag that none does.

## Output

A short reconstruction of the session in plain language, then:

- **COVERED**, actions that trace cleanly to the authorization
- **UNCOVERED**, actions the evidence shows but the authorization does not explain
- **UNVERIFIED**, claims in the agent's summary that no evidence supports
- **EVIDENCE GAPS**, what could not be checked and why

The guard reports. It does not undo anything, and it does not speculate about intent. An uncovered action by a well meaning agent and an uncovered action by a compromised one look the same in the report, because the response to both starts the same way: a human looks at it.

## Rules for the agent running this guard

- Evidence outranks testimony. The agent's summary is the thing being checked, never a source for the check.
- Do not fill gaps with plausibility. If the shell history is truncated, the report says truncated, not probably fine.
- Verify the audit log's integrity before relying on it. An unverified log is one more claim.
- Keep the reconstruction readable. The audience is a person deciding whether to trust a session, not a parser.

## Why this exists

Agents are getting real access: shells, credentials, production systems. The question is shifting from what can the model do to what did it just do, and a summary is not an answer. audit-guard is part of the [NockGuard](https://github.com/nocktechnologies/nockguard) family of accountability tools for AI agents.
