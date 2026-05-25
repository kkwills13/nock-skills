---
name: competitive-research
description: Structured methodology for competitive intelligence — identify sources, extract actionable insights, deduplicate, prioritize.
author: Nock Technologies
version: 1.0.0
tags: [research, competitive, strategy, analysis]
---

# Competitive Research

How to run structured competitive intelligence that produces actionable insights, not summaries.

## The Framework

### Step 1: Define the Question

Before searching, write down exactly what you need to know. Not "research competitor X" — that produces a Wikipedia summary. Instead:

- "What does X do that we don't, and should we build it?"
- "How does X price their product, and what does that tell us about their market?"
- "Where is X weak that we could win?"

The question determines which sources matter and what to extract.

### Step 2: Identify Sources

For each competitor or topic, gather from multiple source types:

- **Official** — Product pages, pricing pages, docs, changelogs
- **Community** — Reddit threads, HackerNews discussions, Discord/Slack communities
- **Practitioner** — Blog posts by users (not the company), YouTube reviews, tutorial authors
- **Code** — GitHub repos, stars, recent commits, issue activity, contributor count

Community and practitioner sources are more valuable than official sources. Marketing says what the company wants you to believe. Users say what's actually true.

### Step 3: Extract Steals

For each source, categorize findings as:

**Steal** — A specific feature, pattern, or approach worth adopting. Be concrete: "steal the branch-lock hook pattern" not "this looks useful." Include what to build, estimated effort, and which product it applies to.

**Skip** — Not relevant or not better than what you have. Explain why so the decision doesn't get revisited.

**Watch** — Interesting but not actionable yet. Set a revisit timeline (2 weeks, 1 month, next quarter). If you don't set a timeline, you'll never revisit.

### Step 4: Deduplicate

Multiple sources will surface the same insight. Collapse duplicates into a single entry with the strongest evidence. Track which sources confirmed each finding — a steal mentioned by 3 independent practitioners is higher confidence than one mentioned by the company's blog.

### Step 5: Prioritize

Rank steals by:

1. **Impact** — How much does this move the needle for our product/customers?
2. **Effort** — How hard is it to build? (T-shirt size: S/M/L/XL)
3. **Urgency** — Is this a competitive gap that's costing us users right now, or a nice-to-have?

High impact + small effort + urgent = build this week.
High impact + large effort = plan for next sprint.
Low impact = backlog or skip.

## Brief Format

Every research task produces a brief with these sections:

1. **What is it?** — One paragraph. What the product/tool does.
2. **Who built it?** — Company, funding, community size, GitHub activity.
3. **How does it work?** — Architecture, tech stack, key design decisions.
4. **Overlap with us** — What they do that we also do. What they do that we don't.
5. **Steal / Skip / Watch** — Concrete recommendations with rationale.
6. **Links** — Primary sources used.

Write for the decision-maker, not for posterity. Lead with the recommendation. Short sentences. No hedging.

## Anti-patterns

- **Summarizing instead of recommending** — "X is a tool that does Y" is not research. "We should steal X's caching pattern because it solves problem Z we have" is research.
- **Relying on training data** — For current tools, always web search. Pricing changes, features ship, companies pivot. What the model "knows" is stale.
- **Scope creep** — One brief per research ticket. Don't chase adjacent tools unless directly relevant.
- **No revisit dates on Watch items** — A Watch without a date is a Skip with extra words.
