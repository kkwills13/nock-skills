# nock-skills

Claude Code skills from [Nock Technologies](https://nocktechnologies.com) — patterns extracted from running autonomous AI development pipelines at production scale.

Each skill is a folder with a `SKILL.md` file that Claude Code can load as a slash command or via the Skill tool.

## Skills

| Skill | What it does |
|-------|-------------|
| [engineering-pipeline](./engineering-pipeline/) | 7-phase PR pipeline — plan, build, verify, gate, push, review, merge+docs |
| [session-handoffs](./session-handoffs/) | Structured context transfer between sessions |
| [branch-isolation](./branch-isolation/) | Branch discipline for parallel agents |

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

These skills were extracted from Nock Technologies' internal Claude Code pipeline, which runs Kit (Claude) and Codex as parallel autonomous agents building [NockCC](https://nocktechnologies.io) — a multi-agent AI development dashboard.

The patterns here solve real problems we ran into at scale:
- **engineering-pipeline** — prevents the most common agent mistake: shipping code without a security review or docs update
- **session-handoffs** — prevents context loss between the ~71-hour session limit resets
- **branch-isolation** — prevents agents on parallel branches from accidentally switching to each other's branch and contaminating diffs

## License

MIT — use freely, attribution appreciated.

---

Built by [Nock Technologies](https://nocktechnologies.com) · [nocktechnologies.io](https://nocktechnologies.io)
