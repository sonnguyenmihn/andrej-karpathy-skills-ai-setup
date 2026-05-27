# Adapter: Claude Code

> How to wire this `ai-setup/` template into Claude Code (CLI + IDE).

## Quick install (5 minutes)

For a new project:

```bash
# 1. Copy the Karpathy pitfalls into CLAUDE.md
cp ai-setup/frameworks/02-coding-pitfalls.md ./CLAUDE.md

# 2. Append the most important rules
for f in 01-think-before-coding 02-simplicity-first 03-surgical-changes 04-goal-driven-execution 05-context-engineering; do
  echo "" >> ./CLAUDE.md
  echo "---" >> ./CLAUDE.md
  echo "" >> ./CLAUDE.md
  cat "ai-setup/rules/${f}.md" >> ./CLAUDE.md
done

# 3. Seed the memory files
cp ai-setup/memory/PROJECT.md.template ./PROJECT.md
cp ai-setup/memory/DECISIONS.md.template ./DECISIONS.md
cp ai-setup/memory/PROGRESS.md.template ./PROGRESS.md
cp ai-setup/memory/GOTCHAS.md.template ./GOTCHAS.md

# 4. Create a skills/ folder for project-specific skills
mkdir -p skills/
cp ai-setup/skills/SKILL.md.template skills/_example.SKILL.md.template
```

## CLAUDE.md philosophy for Claude Code

Claude Code reads `CLAUDE.md` files at every turn — meaning every byte costs context window space. So:

- **Don't dump everything in CLAUDE.md.** Put stable, universal rules here. Move task-specific knowledge into skills.
- **Layer CLAUDE.md files.** Repo-level `CLAUDE.md` for project-wide rules. Subdirectory `CLAUDE.md` files for area-specific rules (`backend/CLAUDE.md`, `frontend/CLAUDE.md`).
- **Treat CLAUDE.md as a living budget.** Periodically audit: is every line here earning its keep?

A reasonable CLAUDE.md size: **1–3KB total** for most projects. Larger only if you really need it.

## Mapping concepts to Claude Code primitives

| `ai-setup/` concept | Claude Code primitive |
|---|---|
| Rules | `CLAUDE.md` |
| Skills | `~/.claude/skills/<name>/SKILL.md` (global) or `skills/<name>/SKILL.md` (project) |
| Memory files | Plain markdown files at repo root, referenced from `CLAUDE.md` |
| Subagents | Built-in subagent invocation; declared in skills or invoked ad-hoc |
| Routines | `/schedule` / `/loop` commands; the Routines feature |
| Hooks | `~/.claude/hooks.json` or project-level hook config |
| MCP tools | MCP server config in `~/.claude.json` or project mcp config |
| Worktrees | Standard `git worktree`; Claude Code respects them naturally |

## Recommended workflow

### Daily

1. Start the session with `/resume` if continuing prior work, or fresh otherwise.
2. Before substantive work, run [`prompts/plan.md`](../prompts/plan.md).
3. Work in **auto mode** for parallel agents; **manual mode** for sensitive operations.
4. End the session with [`prompts/reflect.md`](../prompts/reflect.md) — updates `PROGRESS.md`, `DECISIONS.md`, `GOTCHAS.md`.

### Weekly

1. Read 3 random session transcripts. Look for patterns the agent should know but doesn't.
2. Audit `CLAUDE.md` size. Anything to move to skills?
3. Check cache hit rate (Anthropic Console). Trending down?
4. Run a routine: "Read the past week of sessions. Update `GOTCHAS.md` with anything new. Show me the diff."

### Monthly

1. Audit your skills folder. Any unused? Any duplicated? Any to split?
2. Audit your routines. Still serving their purpose? Anything new to automate?
3. Update evals with any new failure modes seen in production.

## Setting up worktrees for parallel work

```bash
# Create a worktree per workstream
git worktree add wt-feature-a feature/a
git worktree add wt-feature-b feature/b

# Run claude in each, with distinct session names and colors
cd wt-feature-a && claude
# /name feature-a
# /color blue

cd wt-feature-b && claude
# /name feature-b
# /color green
```

Both agents share the same repo, the same `CLAUDE.md`, the same skills. They don't share working memory unless you explicitly point them at shared files.

## Setting up auto mode

Read the auto-mode docs for the latest. Generally:

```bash
claude --auto
```

Auto mode runs a classifier + adversarial check before each tool call. ~30–40% more tokens, but unlocks overnight and parallel work without approval prompts.

**Combine with sandboxing**: prefer worktrees for the agent's working directory; don't give auto-mode agents direct production access.

## Routines

Use the `/schedule` command to create a routine. Examples:

```
/schedule daily 09:00 "Read recent PRs merged to main. Update docs that drift from code. Open a PR if any updates needed."

/schedule weekly mon 10:00 "Scan dependencies for security advisories. Open PRs for safe patches. Skip major version bumps."

/schedule on-event github.pr.opened "If CI fails on style or trivial test issues, fix and re-push. Don't fix anything substantive."
```

See [`components/05-orchestration.md`](../components/05-orchestration.md) for the conceptual treatment.

## Plugins

If you want to publish parts of this setup as Claude Code plugins:

- `.claude-plugin/plugin.json` in this repo already exists — see the marketplace config.
- A plugin = a set of skills + optional MCP server + optional hooks.
- Daisy's note: plugins **cannot** ship a CLAUDE.md (intentionally). If you need to inject context unconditionally, use a session-start hook.

## Reading guide

- [`components/`](../components/) — each file points to the relevant transcripts.
- [`../transcripts/code-with-claude-2026/london/02-beyond-the-basics-with-claude-code.md`](../../transcripts/code-with-claude-2026/london/02-beyond-the-basics-with-claude-code.md) — the canonical talk on Claude Code customization.
- This repo's main `README.md` for the published-skills story.
