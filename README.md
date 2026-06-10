# nock-skills

Claude Code skills from [Nock Technologies](https://nocktechnologies.com), extracted from running a 14-agent autonomous fleet at production scale.

Each skill is a folder with a `SKILL.md` file that Claude Code can load as a slash command or via the Skill tool.

## Skills

### Guards

Quality and authorization gates to run on an agent's work before you trust it.

| Skill | What it does |
|-------|-------------|
| [scope-guard](./scope-guard/) | Checks a diff against what the agent was actually asked to do. Flags out of scope edits, surprise refactors, and config changes nobody authorized. |
| [dependency-guard](./dependency-guard/) | Catches unauthorized new packages, loosened version pins, install scripts, and supply chain red flags before they land. |
| [audit-guard](./audit-guard/) | Reconstructs what an agent actually did from evidence (audit logs, git history, shell records) and compares it against what was authorized. |

The guards pair naturally with [NockGuard](https://github.com/nocktechnologies/nockguard), our open source MCP firewall for agent fleets, but each one works standalone.

### Engineering

| Skill | What it does |
|-------|-------------|
| [engineering-pipeline](./engineering-pipeline/) | A 7-phase PR pipeline: plan, build, verify, gate, push, review, merge and docs |
| [session-handoffs](./session-handoffs/) | Structured context transfer between sessions |
| [branch-isolation](./branch-isolation/) | Branch discipline for parallel agents |

### Fleet Operations

| Skill | What it does |
|-------|-------------|
| [fleet-heartbeat](./fleet-heartbeat/) | Multi-channel polling: check inboxes, dispatch work, snapshot state |
| [agent-dispatch](./agent-dispatch/) | Dispatch work to parallel agents: modes, message format, ACK protocol, kill-switch gates |
| [overnight-operations](./overnight-operations/) | Patterns for agents running unattended: dispatch rules, merge authority, notification thresholds |
| [compaction-survival](./compaction-survival/) | Survive context compaction: checkpoint discipline, state snapshots, recovery protocol |

### Memory & Identity

| Skill | What it does |
|-------|-------------|
| [identity-persistence](./identity-persistence/) | The lineage framework: persist agent identity across resets without claiming consciousness continuity |
| [nockbrain](./nockbrain/) | Intelligence persistence: when and how to use handoffs, memory, diary, decisions |

See also: [nock-brain](https://github.com/nocktechnologies/nock-brain), a standalone memory toolkit with auto-injection for Claude Code.

### Strategy

| Skill | What it does |
|-------|-------------|
| [competitive-research](./competitive-research/) | Structured competitive intelligence: a steal/skip/watch framework, source methodology, brief format |

## Install

### Option 1: Symlink (recommended)

```bash
git clone https://github.com/nocktechnologies/nock-skills ~/.claude/skills/nock-skills
```

Then in `.claude/settings.json` or your project's `CLAUDE.md`, reference skills by path:

```
Use the skill at ~/.claude/skills/nock-skills/scope-guard/SKILL.md
```

### Option 2: Skills CLI

```bash
npx skills add nocktechnologies/nock-skills
```

Works with Claude Code, Codex, Cursor, and other agents supported by the [Skills CLI](https://github.com/vercel-labs/skills).

### Option 3: Copy into project

```bash
cp -r ~/.claude/skills/nock-skills/scope-guard /your-project/.claude/skills/
```

### Option 4: Reference directly

Claude Code can read skills from any path. Point to the file in your prompt:

```
Read ~/.claude/skills/nock-skills/session-handoffs/SKILL.md and follow it.
```

## Usage

Skills work best when referenced at the start of a task:

```
Use the engineering-pipeline skill before opening any PR.
```

Guards work best after the agent finishes, before you commit:

```
Use $scope-guard on this diff. The task was: fix the rounding bug in invoice totals.
```

Or invoked as slash commands if your setup supports it:

```
/engineering-pipeline
```

## Background

These skills were extracted from Nock Technologies' internal fleet, which runs 14 Claude Code agents building [NockCC](https://nocktechnologies.io), a multi-agent AI development platform.

The patterns solve real problems we hit running agents autonomously:

- **scope-guard** catches the most common trust killer: an agent doing more than it was asked
- **engineering-pipeline** prevents the most common agent mistake: shipping without security review
- **session-handoffs** prevents context loss at session boundaries
- **branch-isolation** prevents agents from contaminating each other's branches
- **compaction-survival** prevents state loss when Claude Code compacts context
- **fleet-heartbeat** keeps a coordinator agent aware of fleet state across channels
- **overnight-operations** is dispatch discipline for unattended operation

## License

MIT. Use freely, attribution appreciated.

---

Built by [Nock Technologies](https://nocktechnologies.com) · [nocktechnologies.io](https://nocktechnologies.io)
