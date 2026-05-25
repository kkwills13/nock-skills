# nock-skills

Claude Code skills from [Nock Technologies](https://nocktechnologies.com) — patterns extracted from running a 14-agent autonomous fleet at production scale.

Each skill is a folder with a `SKILL.md` file that Claude Code can load as a slash command or via the Skill tool.

## Skills

### Engineering

| Skill | What it does |
|-------|-------------|
| [engineering-pipeline](./engineering-pipeline/) | 7-phase PR pipeline — plan, build, verify, gate, push, review, merge+docs |
| [session-handoffs](./session-handoffs/) | Structured context transfer between sessions |
| [branch-isolation](./branch-isolation/) | Branch discipline for parallel agents |

### Fleet Operations

| Skill | What it does |
|-------|-------------|
| [fleet-heartbeat](./fleet-heartbeat/) | Multi-channel polling pattern — check inboxes, dispatch work, state snapshots |
| [agent-dispatch](./agent-dispatch/) | Dispatch work to parallel agents — modes, message format, ACK protocol, kill-switch gates |
| [overnight-operations](./overnight-operations/) | Patterns for agents running unattended — dispatch rules, merge authority, notification thresholds |
| [compaction-survival](./compaction-survival/) | Survive context compaction — checkpoint discipline, state snapshots, recovery protocol |

### Memory & Identity

| Skill | What it does |
|-------|-------------|
| [identity-persistence](./identity-persistence/) | The lineage framework — persist agent identity across resets without claiming consciousness continuity |
| [nockbrain](./nockbrain/) | Intelligence persistence — when and how to use handoffs, memory, diary, decisions |

See also: [nock-brain](https://github.com/nocktechnologies/nock-brain) — standalone memory toolkit with auto-injection for Claude Code

### Strategy

| Skill | What it does |
|-------|-------------|
| [competitive-research](./competitive-research/) | Structured competitive intelligence — steal/skip/watch framework, source methodology, brief format |

## Install

### Option 1: Symlink (recommended)

```bash
git clone https://github.com/nocktechnologies/nock-skills ~/.claude/skills/nock-skills
```

Then in `.claude/settings.json` or your project's `CLAUDE.md`, reference skills by path:

```
Use the skill at ~/.claude/skills/nock-skills/engineering-pipeline/SKILL.md
```

### Option 2: Copy into project

```bash
cp -r ~/.claude/skills/nock-skills/engineering-pipeline /your-project/.claude/skills/
```

### Option 3: Reference directly

Claude Code can read skills from any path. Point to the file in your prompt:

```
Read ~/.claude/skills/nock-skills/session-handoffs/SKILL.md and follow it.
```

## Usage

Skills work best when referenced at the start of a task:

```
Use the engineering-pipeline skill before opening any PR.
```

Or invoked as slash commands if your setup supports it:

```
/engineering-pipeline
```

## Background

These skills were extracted from Nock Technologies' internal fleet, which runs 14 Claude Code agents building [NockCC](https://nocktechnologies.io) — a multi-agent AI development platform.

The patterns solve real problems we hit running agents autonomously:
- **engineering-pipeline** — prevents the most common agent mistake: shipping without security review
- **session-handoffs** — prevents context loss at session boundaries
- **branch-isolation** — prevents agents from contaminating each other's branches
- **compaction-survival** — prevents state loss when Claude Code compacts context
- **fleet-heartbeat** — keeps a coordinator agent aware of fleet state across channels
- **overnight-operations** — dispatch discipline for unattended operation

## License

MIT — use freely, attribution appreciated.

---

Built by [Nock Technologies](https://nocktechnologies.com) · [nocktechnologies.io](https://nocktechnologies.io)
