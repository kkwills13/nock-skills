---
name: "engineering-pipeline"
description: "7-phase PR pipeline for Claude Code agents: plan, build, verify, gate, push, review, merge+docs. Use this before starting any non-trivial feature or fix. Ensures security review, adversarial audit, and docs are never skipped."
---

# Engineering Pipeline

A structured 7-phase pipeline for shipping code with Claude Code. Each phase is a gate. Don't skip forward.

## When to use

Start this pipeline whenever you're about to open a PR. The 7 phases keep quality high and prevent the most common agent mistakes: skipping tests, skipping security review, forgetting docs.

## Phase 1: Plan

Before writing any code:

1. Read the spec or prompt carefully. If anything is ambiguous, ask before proceeding.
2. Identify which files will change and why.
3. Write a one-paragraph implementation plan. If the change touches auth, networking, file I/O, or subprocess execution, flag it explicitly.
4. Check for existing patterns in the codebase before introducing a new one.

**Output:** A clear mental model of what you're building and where.

## Phase 2: Build

- Work in incremental vertical slices: one end-to-end piece at a time, not horizontal layers.
- Write the test first (TDD), then the implementation. Red → Green → Refactor.
- Keep commits small and focused. Each commit should be independently reviewable.
- Load only the context you need for the current slice. Don't try to hold the whole codebase in mind.

**Output:** Working code with tests passing.

## Phase 3: Verify

Run all three in order:

1. **Security review**: scan for: path traversal, command injection, hardcoded secrets, unsafe deserialization, missing auth checks, SSRF. Required if you touched: file I/O, subprocess, network calls, auth, environment variables.
2. **Code simplification**: remove dead code, redundant abstractions, and unnecessary complexity. If a function does two things, split it.
3. **Code review**: check for: logic errors, missing edge cases, inconsistent naming, missing tests for new behavior.

**Output:** Clean, secure code with no obvious issues.

## Phase 4: Adversarial Gate

Stop. Before pushing:

1. Run an adversarial audit: ask a fresh context (or a second agent) to find problems you missed.
2. Review the audit output. Address **all** findings before proceeding.
3. If findings were significant, re-run Phase 3.

This is the phase most commonly skipped. Don't skip it.

**Output:** Audit findings addressed. You are confident this is ready for human eyes.

## Phase 5: Push

1. Push to a feature branch (never directly to main/master).
2. Open a PR with:
   - A clear title (what changed, not how)
   - A summary: what, why, and any deployment notes
   - A test plan: how to verify the change works
3. Let automated reviewers run (CI, CodeRabbit, Copilot, Gemini, etc.).

**Output:** PR open, CI running.

## Phase 6: Human Review

Wait for:
- Automated reviews to complete
- Product/domain owner to approve the change
- Architecture/strategy owner to approve the approach

Don't self-merge unless explicitly authorized.

**Output:** PR approved by the right people.

## Phase 7: Merge + Docs

After merge:

1. Update `CLAUDE.md` if repo structure or conventions changed.
2. Update `CHANGELOG.md` with what shipped.
3. Update `README.md` if there are user-facing changes.
4. Update architecture diagrams if new modules or data flows were introduced.

**Output:** Docs reflect the current state of the codebase.

## Review priority (when addressing reviewer feedback)

Fix immediately:
- Security vulnerabilities
- Bugs and logic errors
- Test failures or missing coverage

Fix if it's clean and quick:
- Performance suggestions
- Better naming
- Missing documentation on public APIs

Use judgment (don't block the PR on these):
- Style preferences that conflict with existing patterns
- Speculative "what if" refactors
- Cosmetic issues in test code
