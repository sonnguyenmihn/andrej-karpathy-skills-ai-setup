# Component: Skills

> **A folder + a `SKILL.md`. The agent loads the body on demand. The middle abstraction between "everything in CLAUDE.md" and "wrap it in MCP."**

## Anatomy of a skill

```
skills/
└── deploy-staging/
    ├── SKILL.md          ← always-loaded one-line description in frontmatter
    ├── deploy.sh         ← script the skill calls
    ├── rollback.sh
    └── examples/
        └── last-week.md  ← examples the skill body references
```

`SKILL.md` structure:

```markdown
---
name: deploy-staging
description: Deploy the current branch to staging. Use when the user wants to ship, deploy, or push to staging.
---

# Deploy to Staging

## When to use this
- The user explicitly asks to deploy / ship / push.
- The branch passes CI.

## Steps
1. Run `./deploy.sh --env staging`
2. Verify https://staging.example.com responds 200
3. Post to #deployments

## Failure recovery
If the deploy errors out, run `./rollback.sh` immediately.
```

The frontmatter `description` is the one-liner that lives in the system prompt. The body is loaded only when the agent decides this skill is relevant.

## Why skills (instead of alternatives)

| Alternative | Problem skills solve |
|---|---|
| Everything in `CLAUDE.md` | You pay for it every turn whether you use it or not |
| MCP server wrapping your own CLI | Auth setup, lifecycle, transport — overkill if you have a shell |
| Inline-prompted instructions | Forgotten across sessions; not reusable |
| A scattered `scripts/` folder | Agent doesn't know what's in there or when to use it |

Daisy Hollman:

> "If you already have a CLI, it doesn't make a whole lot of sense to wrap that CLI in MCP unless you're shipping it to non-technical customers. Usually a skill that just tells Claude how to use the CLI is much easier to write up."

## What makes a good skill

### Single, clear purpose

One skill = one capability. If the description has "and" in it twice, split it.

Bad: `database-migrations-and-seeds-and-rollbacks`
Good: `run-migrations`, `seed-database`, `rollback-migration` — three skills.

### Triggerable from description alone

The model decides whether to load the skill based on the description in the system prompt. If the description is too vague, the skill won't trigger when it should.

Test: read just the description. Can you tell when this skill applies?

Bad: `Helpful database stuff.`
Good: `Run pending Django migrations against the current environment. Use when the user wants to migrate, apply migrations, or upgrade the schema.`

Daisy notes: reliably triggering a skill sometimes takes up to a paragraph in the description. Don't be afraid of paragraph-length descriptions for important skills.

### Self-contained body

The body should let the agent do the thing without external lookup:
- Scripts in the skill folder.
- Step-by-step instructions.
- Examples (especially of the output format).
- Recovery instructions for common failures.

If the body says "ask Alice for the deploy key," you've failed the self-contained test.

### Scales with intelligence

Recall the "tools that scale with intelligence" principle (Daisy). A skill that says "Use this exact 17-step procedure or it will break" doesn't scale — better models will be locked into a procedure that may not apply. A skill that says "Here are the principles and the gotchas; figure out the right steps for this situation" scales.

## Anti-patterns

### The mega-skill

A 50-page `SKILL.md` that tries to be a manual for your whole product. The model loads it and the context window dies.

Fix: split into smaller skills with clear, non-overlapping purposes.

### The micro-skill

50 skills that each do one tiny thing. The frontmatter descriptions alone fill the system prompt.

Fix: consolidate related capabilities under a few "verb-noun" skills with multiple sub-procedures inside.

### The skill nobody triggers

You wrote it. The model never loads it. The description was too vague or the use case was too narrow.

Fix: rewrite the description until reading it cold tells you exactly when it applies. If nobody triggers it, maybe the use case doesn't justify a skill.

### The skill that duplicates `CLAUDE.md`

If your skill body is the same content you already have in `CLAUDE.md`, one of them is wrong. Project conventions belong in project memory (`CLAUDE.md` / rules / `AGENTS.md`); skills are _capabilities_, not _conventions_.

## When to extract content into a skill

| Trigger | Action |
|---|---|
| You explained the same procedure 3+ times across sessions | Make it a skill |
| There's a script the agent should know to run but doesn't | Wrap the script in a skill |
| `CLAUDE.md` has a section that only matters in specific tasks | Move it to a skill so it loads on demand |
| A workflow has 5+ steps and the model frequently misses one | Skill |
| The user has to remember to invoke a particular tool | Skill (let the model decide based on description) |

## A note on hierarchy

As of late 2025 / early 2026: skill hierarchies (sub-skills, skill packs) are not yet well-supported by most tools. Don't design for hierarchies that don't exist. Daisy hinted at upcoming support; until it lands, design flat.

## Hosting and versioning

A skill is just files. You can:
- Commit it to the project repo (`skills/<name>/`). Best for project-specific skills.
- Maintain a shared skills repo across projects (this repo is one such pattern).
- Distribute via your tool's plugin system (Claude Code plugins, Cursor's rule-sharing, etc.).

`git` is your versioning. Treat skills like code.

## Template

See [`../skills/SKILL.md.template`](../skills/SKILL.md.template) for a starter you can copy.

## Reading guide

- Daisy Hollman, _Beyond the basics with Claude Code_ (London) — the canonical talk on plugin abstractions including skills.
- Mahesh, _Memory and dreaming_ (SF) — skills as procedural memory.

→ [`../transcripts/code-with-claude-2026/london/02-beyond-the-basics-with-claude-code.md`](../../transcripts/code-with-claude-2026/london/02-beyond-the-basics-with-claude-code.md)
