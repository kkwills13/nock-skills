---
name: dependency-guard
description: Use after an agent changes a project to catch new or modified dependencies that nobody authorized. Flags new packages, loosened version pins, install scripts, and supply chain red flags before they land.
---

# dependency-guard

The fastest way for an agent to solve a problem is to install something. Sometimes that is the right call. Sometimes it pulls an unvetted dependency tree into your build, swaps a pinned version for a floating one, or grabs a package whose name is one letter off from the real thing.

This guard reviews the dependency changes in a diff and reports what changed, whether it was authorized, and what risk it carries.

## When to use

Run it after your agent finishes, before anything is committed:

```text
Use $dependency-guard on this diff.
Use $dependency-guard on this diff. The task allowed adding a CSV parsing library.
```

If the task authorized a specific addition, say so. Everything else gets flagged.

## What it checks

1. **New packages.** For each addition: was it asked for, is the name spelled exactly right (typosquats live one character away from popular packages), how established is it, and does it carry install hooks.
2. **Version pin changes.** Pinned to floating, broadened ranges, and silent downgrades are all findings. A loosened pin is how a future install pulls in something nobody reviewed.
3. **Manifest and lockfile agreement.** The manifest and the lockfile must tell the same story. A lockfile change with no manifest change, or the reverse, is a finding.
4. **Install scripts.** postinstall hooks and their equivalents in other ecosystems run code on every machine that installs the project. Any new or changed install script is flagged for human review.
5. **Source changes.** A dependency moved from the public registry to a git URL, a custom registry, or a vendored archive changes who controls that code. Always a finding.

## Output

One entry per dependency change:

- **AUTHORIZED**, the task named it and the package checks out
- **UNAUTHORIZED**, nobody asked for this, with the package, the version, and what it brings with it
- **SUSPICIOUS**, authorized or not, something about the package itself warrants a human look: typosquat distance, install hooks, very new publish date, or an unusual source

If there are no dependency changes in the diff, the guard says exactly that in one line.

## Rules for the agent running this guard

- Check the actual registry when you can. A package's publish date, maintainer count, and weekly downloads are evidence. Your memory of the package is not.
- Spell-check against the popular package the name resembles. `requets`, `lodahs`, and friends are attacks, not typos to autocorrect.
- Treat transitive additions seriously. A small direct dependency that pulls in forty packages is a forty package decision.
- Never approve your own addition. If the same session both added the dependency and is running this guard, flag it for a human regardless.

## Why this exists

Most dependency review happens at install time, which is after the code already ran on a developer machine. This guard moves the question earlier: did anyone actually authorize this, and would a careful human say yes. dependency-guard is part of the [NockGuard](https://github.com/nocktechnologies/nockguard) family of accountability tools for AI agents.
