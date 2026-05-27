# Component: Memory

> **What the agent knows across turns and sessions. The thing that makes agents stop re-learning every morning.**

## Three layers, each with different access patterns

| Layer | Lifetime | Update frequency | Example |
|---|---|---|---|
| **Working memory** | Single task / session | Often, by the agent itself | A `NOTES.md` file the agent edits while debugging |
| **Project memory** | Whole project | Occasionally, mostly by humans | `CLAUDE.md`, `.cursor/rules/`, `AGENTS.md`, `PROJECT.md` |
| **Knowledge base** | Long-lived, sometimes shared | Curated, often via batch processes | Org-wide runbooks, decision logs, distilled past sessions |

Keep them separate. Mixing layers is the #1 way memory turns into context-window bloat.

## File-system memory has won

Frontier models are good at managing files with `bash`, `grep`, `cat`, `sed`. Mahesh from Anthropic (_Memory and dreaming for self-learning agents_):

> "We know that agents are able to manage a virtual environment and manage their own file system. Why can't we go the same direction with memory?"

This is the default in Claude's managed agents and the implicit pattern in Claude Code (CLAUDE.md, scratch files), Cursor (rules, notes), and almost every successful agent system.

**Implication**: don't build a vector DB you don't need. Don't build a memory API you don't need. Files in folders, with clear naming, get you 90% of the way.

When you _do_ need structured memory (semantic search across millions of items), add it as a tool the agent calls — not as a substitute for the file-system memory.

## The four canonical files

This template includes four memory file templates. Use them, customize them, or replace them — but cover the four roles.

| File | Role | Update cadence |
|---|---|---|
| `PROJECT.md` | What this project is, scope, constraints, audience | Rare — when scope changes |
| `DECISIONS.md` | ADR-lite. Decisions made, the why, the alternatives | Per decision |
| `PROGRESS.md` | Current state, what's in flight, where you stopped | Every session |
| `GOTCHAS.md` | Non-obvious things, lessons learned, anti-patterns specific to this codebase | When you hit one |

See [`../memory/`](../memory/) for the templates.

**Why these four**: they map to the questions humans ask when they join a project:
- _What is this thing?_ → `PROJECT.md`
- _Why is it this way?_ → `DECISIONS.md`
- _Where are we?_ → `PROGRESS.md`
- _What's going to bite me?_ → `GOTCHAS.md`

If you can't answer those four questions in under 5 minutes from your memory files, the agent can't either.

## Concurrency, versioning, attribution

If multiple agents share memory (you _will_ get here as you scale orchestration):

- **Optimistic concurrency**: use a content hash before overwriting. Anthropic's managed agents do this; you can DIY with a simple lockfile or `git`.
- **Version history**: every memory update should be audit-able. `git` is the simplest tool for this. Use it.
- **Attribution metadata**: which agent / session made which update. Header comments in files work.
- **Permission scopes**: not every agent should write to every memory store. Read-only for shared knowledge, read-write for working memory.

## "Dreaming" — batch memory consolidation

Anthropic's term, but the concept is general: a **batch out-of-band process** that reads recent session transcripts and updates memory.

What it does:
- Finds patterns across sessions a single agent wouldn't notice.
- Deduplicates similar entries.
- Removes stale entries.
- Adds verification notes ("at this date, this memory was confirmed accurate").

When to run:
- Nightly (cron-style).
- After significant work (post-sprint, post-incident).
- When you notice memory drifting from reality.

A simple DIY version: a script that runs `claude` / `cursor` / your tool with a prompt like:

```
Read all session transcripts from the past 7 days under .ai/sessions/.
Update memory/GOTCHAS.md with any new lessons.
Update memory/DECISIONS.md with any new decisions.
Mark any obviously stale entries with [STALE - YYYY-MM-DD].
Show me a diff for review.
```

This isn't fancy. It's a routine. (See [`05-orchestration.md`](05-orchestration.md) for routines.)

## What to put in memory, what to leave out

In memory:
- Project conventions ("we use snake_case in Python, camelCase in TS").
- Decisions and their rationale ("we chose Redis over Memcached because of persistence requirements").
- Gotchas ("the `migrate` command must run before `seed` or it silently corrupts the DB").
- Pointers to where things live ("billing logic is in `src/billing/`, owner is @alice").

Not in memory:
- Things that change every turn (timestamps, request IDs).
- Things easily looked up (don't memorize file contents; the agent can `cat` the file).
- Generic best practices (those live in rules / skills).
- Anything secret (memory is not a secrets store).

## Anti-patterns

- **One enormous `CLAUDE.md`.** Every line is paid for every turn. Move things into skills or `DECISIONS.md` so they're loaded only when relevant.
- **No `GOTCHAS.md`.** You'll re-learn the same lessons every Monday.
- **Vector DB for everything.** Premature optimization. File system + grep + read first; add retrieval only when you have a real retrieval problem.
- **Memory writes with no review.** Especially for long-lived knowledge bases — let a human (or a "dreaming" job) curate before things land in shared memory.
- **Treating memory as transcript.** Memory should be _distilled_. If you find yourself reading raw chat history, your memory layer isn't doing its job.

## Reading guide

- Mahesh, _Memory and dreaming for self-learning agents_ (SF + London) — file-system memory, dreaming, multi-agent concurrency.
- Daisy Hollman, _Beyond the basics with Claude Code_ (London) — memory vs. plugin abstractions; "memory has its place… low quality, low cost, short-lived information."

→ [`../transcripts/code-with-claude-2026/sf/18-memory-and-dreaming-for-self-learning-agents.md`](../../transcripts/code-with-claude-2026/sf/18-memory-and-dreaming-for-self-learning-agents.md)
→ [`../transcripts/code-with-claude-2026/london/02-beyond-the-basics-with-claude-code.md`](../../transcripts/code-with-claude-2026/london/02-beyond-the-basics-with-claude-code.md)
