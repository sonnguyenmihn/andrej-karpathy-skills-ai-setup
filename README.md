# ai-setup/ — A Reusable Template for AI in Complex Work

> A tool-agnostic template for setting up AI agents on real software projects. Distilled from Andrej Karpathy's commentary on LLM coding and Anthropic's _Code with Claude 2026_ developer conference (36 transcribed sessions in [`../transcripts/code-with-claude-2026/`](../transcripts/code-with-claude-2026/)).

## What this is

Two things:

1. **A human guide** — [`GUIDE.md`](GUIDE.md) — read once, end to end. Everything else here is a more focused version of one section of the guide.
2. **A reusable template** — a directory you can copy into any new project to bootstrap a serious agent setup.

It is **opinionated** and **tool-agnostic**. The concepts work across Claude Code, Cursor, custom agents on the API, and whatever comes next. Adapters for specific tools live in [`adapters/`](adapters/).

## Quick start (10 minutes)

For a fresh project:

```bash
# 1. Read the guide once
$EDITOR ai-setup/GUIDE.md

# 2. Wire up your primary tool
$EDITOR ai-setup/adapters/claude-code.md     # or cursor.md, or generic-agent.md

# 3. Follow the "Quick install" section of your chosen adapter
```

That gets you the Karpathy 4 principles + the most important rules + the four memory files.

## Directory map

```
ai-setup/
├── README.md                   ← you are here
├── GUIDE.md                    ← the human guide (item 1) — start here
│
├── frameworks/                 ← three mental models
│   ├── 01-agent-loop.md        ←   perceive → think → plan → act → verify → reflect
│   ├── 02-coding-pitfalls.md   ←   Karpathy 4 principles
│   └── 03-harness-components.md ←  six harness components
│
├── components/                 ← one file per harness component
│   ├── 01-reasoning.md         ←   model + effort selection
│   ├── 02-memory.md            ←   files / dreaming / multi-agent state
│   ├── 03-context.md           ←   KV cache, tool output shaping, "fixed box"
│   ├── 04-skills.md            ←   skill design and anti-patterns
│   ├── 05-orchestration.md     ←   subagents, worktrees, routines, generator-evaluator-repair
│   └── 06-governance.md        ←   evals, hooks, auto mode, transcripts
│
├── rules/                      ← drop-in markdown rules (tool-agnostic)
│   ├── 01-think-before-coding.md
│   ├── 02-simplicity-first.md
│   ├── 03-surgical-changes.md
│   ├── 04-goal-driven-execution.md
│   ├── 05-context-engineering.md
│   ├── 06-verification-and-evals.md
│   └── 07-prompt-hygiene.md
│
├── prompts/                    ← reusable prompt templates
│   ├── kickoff.md              ←   start a new project
│   ├── plan.md                 ←   plan any non-trivial task
│   ├── debug.md                ←   structured debugging
│   ├── review.md               ←   critic-pattern code review
│   └── reflect.md              ←   end-of-session memory updates
│
├── memory/                     ← memory file templates
│   ├── PROJECT.md.template
│   ├── DECISIONS.md.template
│   ├── PROGRESS.md.template
│   └── GOTCHAS.md.template
│
├── skills/                     ← skill scaffold
│   └── SKILL.md.template
│
└── adapters/                   ← tool-specific wiring
    ├── claude-code.md
    ├── cursor.md
    └── generic-agent.md
```

## How to read this

| If you want… | Start with |
|---|---|
| The full conceptual story | [`GUIDE.md`](GUIDE.md) |
| The mental models | [`frameworks/`](frameworks/) (3 files, ~10 min each) |
| To install on a specific tool | [`adapters/`](adapters/) |
| To fix a specific component (e.g. memory) | [`components/`](components/) (6 files, ~10 min each) |
| Drop-in rules for your project | [`rules/`](rules/) |
| Prompt templates | [`prompts/`](prompts/) |
| To see the source material | [`../transcripts/code-with-claude-2026/`](../transcripts/code-with-claude-2026/) |

## The three frameworks (in 30 seconds)

1. **The agent loop** — every useful agent runs _perceive → think → plan → act → verify → reflect_, fractally nested. Most failures are missing or weak phases.
2. **The four coding pitfalls** (Karpathy) — _think before coding, simplicity first, surgical changes, goal-driven execution_. Defuse the most common LLM coding failure modes.
3. **The six harness components** — _reasoning, memory, context, skills, orchestration, governance_. Design each intentionally.

## The most important non-obvious ideas

From the 36 sessions, these were the ones I kept coming back to. If you only remember five things:

1. **The context window is a fixed box.** Context engineering is the highest-leverage skill, not "bigger context window in 6 months."
2. **Pick abstractions that scale.** Hooks > skills > subagents > MCP > CLAUDE.md, in terms of cost per turn.
3. **Tools that scale with intelligence > tools that compensate for it.** Prefer nudges over hard blocks.
4. **Measure outcomes, not activity.** Survival rate > acceptance rate.
5. **Read your transcripts.** Headline metrics lie.

## Source attribution

Every claim traces to one of:

- **Andrej Karpathy** — the existing `CLAUDE.md` at this repo root, plus his linked posts.
- **The 36 Code with Claude 2026 transcripts** in [`../transcripts/code-with-claude-2026/`](../transcripts/code-with-claude-2026/). Specific talks cited per file.
- **My synthesis** where sources disagree or are silent — marked as such in the file.

If you want the original quote for any claim, the per-component file usually has it.

## A note on caption quality

Most transcripts are YouTube auto-captions or local Whisper transcription. Expect occasional misspellings — "Enthropic" → Anthropic, "clawed" → Claude, lowercased proper nouns. Quality is good enough to follow the content; not good enough to trust as direct quotes without verifying against the video. Direct quotes used in this template have been sanity-checked.

## What's NOT in this template

Intentional omissions:

- **Specific model recommendations.** They change every few months. Use the heuristics in [`components/01-reasoning.md`](components/01-reasoning.md) and re-evaluate.
- **RAG architectures.** Most uses replaceable by good context engineering + skills + file-system memory.
- **Specific MCP server recommendations.** Ecosystem moves fast; the general guidance stays stable.
- **Fine-tuning guidance.** Mostly not worth it for product teams in late 2025 / 2026.

## License

This template inherits the MIT license from the parent repo. Use it, copy it, fork it. Attribution welcome but not required.

---

_Built from the 2026 state of the art. Re-read in 6 months and audit what's still true._
