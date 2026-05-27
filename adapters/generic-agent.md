# Adapter: Generic Agent / Other Tools

> How to wire this `ai-setup/` template into any agent system: OpenAI Codex CLI, Aider, custom agents built on raw APIs, etc.

The template is intentionally tool-agnostic. Every concept here maps to _some_ primitive in your tool. This file gives you the translation table.

## The translation table

| `ai-setup/` concept | Generic name | Where it usually lives |
|---|---|---|
| Rules | System prompt / instructions / "constitution" | A prefix on every model call |
| Skills | On-demand instructions + scripts | A folder loaded into context conditionally |
| Memory files | Working / project / knowledge state | Plain files the agent reads & writes |
| Subagents | Delegated calls with separate context | A tool that invokes another model call |
| Routines | Scheduled / event-triggered agent runs | A cron job, a webhook handler, a background worker |
| Hooks | Side effects on agent events | Wrappers around tool calls and lifecycle events |
| MCP tools | External tools | Whatever tool-calling protocol your tool uses |
| Worktrees | Isolated working dirs | `git worktree` (universal) or just multiple checkouts |
| Auto mode | Reduced approval surface | Whatever permission-bypass mechanism your tool has |
| Plan mode | Forced planning step | A prompt template that requires a plan before code |

## The minimum viable setup

You need three things:

### 1. A system prompt that loads the rules

For any agent that takes a system prompt:

```
You are working in a project. Follow these rules:

[paste content of ai-setup/frameworks/02-coding-pitfalls.md]

[paste content of ai-setup/rules/05-context-engineering.md]

[paste content of ai-setup/rules/06-verification-and-evals.md]

Before any non-trivial work, produce a plan with verifiable success criteria per step (see ai-setup/prompts/plan.md).

The project's current state is in PROJECT.md, DECISIONS.md, PROGRESS.md, and GOTCHAS.md at the project root. Read these before substantive work. Update them after.
```

Keep the system prompt **stable** — same bytes every call — for cache discipline.

### 2. The four memory files at project root

```
PROJECT.md     ← what the project is
DECISIONS.md   ← non-trivial decisions and why
PROGRESS.md    ← current state, what's in flight
GOTCHAS.md     ← lessons learned, non-obvious traps
```

Templates: [`../memory/`](../memory/).

The agent needs file-read and file-write tools to use these. Almost every agent has these.

### 3. A planning prompt for non-trivial tasks

For any task you'd consider "non-trivial" (> 30 minutes of work, > 3 files), kick off with the plan prompt from [`../prompts/plan.md`](../prompts/plan.md).

## Building skill-like loading on tools without skills

If your tool doesn't have a skill primitive, you can DIY:

1. Maintain a `skills/` folder with one subfolder per skill.
2. Maintain a `skills/INDEX.md` listing all skill names + descriptions (one line each).
3. In the system prompt: "Skills are available in `skills/`. Read `skills/INDEX.md` to see what's available. When a task matches a skill description, read the relevant `skills/<name>/SKILL.md` and follow it."

This puts the index in the system prompt (cheap) and loads the body on demand (also cheap — only when the agent decides to read it).

## Hooks

If your agent system supports lifecycle hooks (pre-tool-use, post-tool-use, session-start, etc.):

- **Post-tool-use linter**: run linter / formatter / type-checker after each edit, inject errors back to context.
- **Pre-tool-use guard**: warn before destructive operations.
- **Session-start context**: load `PROJECT.md` + `PROGRESS.md` summary.

If your agent doesn't support hooks: wrap the agent's tool-calling in your own code that does the equivalent. Most agent SDKs let you intercept tool calls.

## Routines without a built-in scheduler

If your tool has no scheduling primitive:

- `cron` + a shell script that runs your agent with a prompt.
- A GitHub Action on a schedule.
- A small service that listens for webhooks and triggers the agent.

The pattern is: _store the prompt, schedule the invocation, give the agent access to the same memory files._

Examples to start with:

```bash
# Daily docs drift check (cron)
0 9 * * * cd /path/to/project && agent-cli run --prompt "Read PRs merged in the past 24h. Update docs that drift from code. Open a PR if changes needed."

# Weekly dep audit
0 10 * * 1 cd /path/to/project && agent-cli run --prompt "Audit dependencies for security advisories. Open PRs for safe patches."
```

## Parallel agents

`git worktree` works everywhere `git` works. Run one agent per worktree:

```bash
git worktree add ../wt-feature-a feature/a
git worktree add ../wt-feature-b feature/b

# In separate terminals
cd ../wt-feature-a && agent-cli ...
cd ../wt-feature-b && agent-cli ...
```

Sharing memory across parallel agents: point them at a shared `memory/` directory outside any single worktree, or symlink shared files in.

## Evals

Most agent tools have no eval framework. Build a tiny one:

```python
# eval.py
TESTS = [
    {"prompt": "...", "expect_contains": "..."},
    {"prompt": "...", "expect_function_called": "..."},
    # ...
]

for t in TESTS:
    result = run_agent(t["prompt"])
    # Score it. Programmatic checks where possible; LLM-judge otherwise.
```

Run on every prompt change. Run on every model change. See [`rules/06-verification-and-evals.md`](../rules/06-verification-and-evals.md).

## What this template assumes about your tool

- ✅ The agent can read and write files.
- ✅ The agent can run shell commands.
- ✅ The agent takes a system prompt or equivalent prefix.
- ✅ You can edit and replay prompts.

If any of these are missing, this template won't fit cleanly. Frontier agent tools all have these; check yours.

## Reading guide

- [`../GUIDE.md`](../GUIDE.md) — the deep dive.
- [`../components/`](../components/) — each component has tool-agnostic content.
- The specific adapter for your tool ([`claude-code.md`](claude-code.md), [`cursor.md`](cursor.md)) — even if you use neither, they have concrete examples to crib from.
