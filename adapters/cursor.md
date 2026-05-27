# Adapter: Cursor

> How to wire this `ai-setup/` template into Cursor.

## Quick install (5 minutes)

For a new project:

```bash
# 1. Create the rules directory if it doesn't exist
mkdir -p .cursor/rules

# 2. Drop the Karpathy pitfalls in as a project rule
cp ai-setup/frameworks/02-coding-pitfalls.md .cursor/rules/01-karpathy-pitfalls.mdc

# 3. Add the substantive rules
cp ai-setup/rules/05-context-engineering.md .cursor/rules/02-context-engineering.mdc
cp ai-setup/rules/06-verification-and-evals.md .cursor/rules/03-verification.mdc
cp ai-setup/rules/07-prompt-hygiene.md .cursor/rules/04-prompt-hygiene.mdc

# 4. Seed the memory files
cp ai-setup/memory/PROJECT.md.template ./PROJECT.md
cp ai-setup/memory/DECISIONS.md.template ./DECISIONS.md
cp ai-setup/memory/PROGRESS.md.template ./PROGRESS.md
cp ai-setup/memory/GOTCHAS.md.template ./GOTCHAS.md
```

## Add frontmatter to Cursor rules

Cursor rule files use `.mdc` extension and support frontmatter for scoping. The simplest "always apply" frontmatter:

```yaml
---
description: <one-line description of when this rule applies>
alwaysApply: true
---
```

For path-scoped rules (e.g. backend-only):

```yaml
---
description: Backend Python conventions
alwaysApply: false
globs: ["backend/**/*.py"]
---
```

Edit each `.mdc` file you copied to add the frontmatter at the top.

## Mapping concepts to Cursor primitives

| `ai-setup/` concept | Cursor primitive |
|---|---|
| Rules | `.cursor/rules/*.mdc` (workspace) or User Rules (global, in Settings → Rules) |
| Skills | No direct primitive — closest is path-scoped rules or referenced markdown |
| Memory files | Plain markdown at repo root; reference from rules with relative paths |
| Subagents | Sub-agent feature in Agent mode |
| Routines | Background agents / scheduled prompts (varies by Cursor version) |
| Hooks | Hooks system — see [`adapters/generic-agent.md`](generic-agent.md#hooks) |
| MCP tools | MCP server config in Cursor Settings |
| Worktrees | Open multiple Cursor windows on different worktrees |
| Modes (Plan/Ask) | Use Plan mode for any non-trivial planning step (replaces your need for a manual plan prompt sometimes) |

## Use Plan Mode

Cursor's Plan mode is the canonical implementation of [the plan-before-code rule](../rules/01-think-before-coding.md). Use it whenever the task is:

- Non-trivial (more than ~3 steps).
- Has multiple valid approaches with tradeoffs.
- Affects multiple files or components.

Plan mode produces a structured plan you can review and approve before any code is written. This _is_ goal-driven execution, baked into the tool.

When in Plan mode, paste [`prompts/plan.md`](../prompts/plan.md) for an extra-thorough plan template if needed.

## User Rules vs Project Rules

Two scopes for rules:

- **User Rules** (Settings → Rules) — global to all your projects. Use for: your personal preferences, the Karpathy pitfalls, general "never do X" rules.
- **Project Rules** (`.cursor/rules/*.mdc`) — checked into git, shared with team. Use for: project conventions, stack-specific rules, architecture decisions.

Recommended split:

| Content | Scope |
|---|---|
| Karpathy 4 pitfalls | User Rules (universal) |
| `05-context-engineering.md` | User Rules (universal) |
| `06-verification-and-evals.md` | User Rules (universal) |
| Project-specific style, stack, conventions | Project Rules |
| `PROJECT.md`, `DECISIONS.md`, `GOTCHAS.md` references | Project Rules |

## Memory files and Cursor

Cursor doesn't have a `CLAUDE.md` equivalent that's auto-loaded. To make `PROJECT.md` / `GOTCHAS.md` available, reference them from a Project Rule:

```yaml
---
description: Project context
alwaysApply: true
---

When working on this project, always consult these files before non-trivial work:
- @PROJECT.md — project scope, constraints, milestone
- @DECISIONS.md — non-trivial decisions and rationale
- @PROGRESS.md — current state and what's in flight
- @GOTCHAS.md — known non-obvious traps

Read @GOTCHAS.md before debugging.
Update @PROGRESS.md at the end of each session.
```

The `@file` syntax pulls the file into context when the rule applies.

## Setting up hooks

Cursor supports hooks for various events. The most useful for this template:

- **Post-edit**: run formatters / linters, feed errors back to agent.
- **Pre-tool-use**: warn on risky operations.
- **Session start**: load project context.

See Cursor's hooks documentation and [`components/06-governance.md`](../components/06-governance.md) for the conceptual story.

## Worktrees with Cursor

Open each `git worktree` as a separate Cursor window. They share the same rules (since rules are in the repo). They have independent chat sessions.

For maximum parallelism with auto-mode-style behavior, use Cursor's background agents — they can work on tasks while you focus on something else.

## Routines / background agents

Cursor's background agents are the rough equivalent of Claude Code routines. They can:
- Run on schedule.
- React to events (PR opened, etc.).
- Operate in a sandboxed environment.

See [`components/05-orchestration.md`](../components/05-orchestration.md#pattern-3-routines-scheduled--event-triggered-agents) for the patterns; Cursor's specific UI for setting them up changes — check the current docs.

## Plugins / extensions

If you want this setup to spread across projects:

- Symlink your User Rules across machines (sync via Dropbox, iCloud, or your dotfiles repo).
- For team sharing, commit the `.cursor/rules/` directory.

## Reading guide

- [`components/`](../components/) — every concept here has a deeper file.
- [`../transcripts/code-with-claude-2026/`](../../transcripts/code-with-claude-2026/) — the source material for most rules.
- Cursor's official docs for the latest on Plan mode, Hooks, and Background Agents.
