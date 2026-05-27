# A Practical Guide to Setting Up AI for Complex Work

> **Read this once, end to end.** Then keep it open as a reference while you wire up your projects. Everything in this `ai-setup/` directory is a more focused version of one section here.

## Who this is for

You're building real software (or running a team that does) and you've decided to use AI agents as a first-class part of your workflow. You're past "neat demo, now what?" and you're hitting the actual hard problems: agents that lose context, prompts that quietly stop working when models change, reviewers who can't keep up, and a vague feeling that you should be _systematic_ about this but no one has shown you what that looks like.

This guide gives you the **mental models, principles, and concrete file structure** to set up AI for non-trivial work. It is tool-agnostic — every concept maps to Claude Code, Cursor, OpenAI Codex, Aider, custom agents built on the API, or whatever comes next — and the `adapters/` directory shows you how.

It is distilled from:

1. **Andrej Karpathy's commentary** on LLM coding pitfalls (the existing `CLAUDE.md` in this repo).
2. **Karpathy's agent-loop framing**: perceive → think → plan → act → verify → reflect.
3. **Anthropic's Code with Claude 2026** developer conference (SF + London, ~17 hrs of talks transcribed in `transcripts/code-with-claude-2026/`).

It is opinionated. Where I disagree with parts of the source material, I say so.

---

## The three frameworks you need in your head

Before any file, prompt, or skill, you need three mental models. They sit at different levels of abstraction and you'll use all three.

### 1. The agent loop (Karpathy): _perceive → think → plan → act → verify → reflect_

Every agent — every _useful_ agent — runs some version of this loop. The phases:

| Phase | What happens | What can go wrong |
|---|---|---|
| **Perceive** | Read the task, the files, the tool outputs, the prior turns | Wrong files loaded, stale context, missed signals |
| **Think** | Reason about state and options (scratchpad / inner monologue) | Hallucinated assumptions, surface-level analysis |
| **Plan** | Choose a sequence of steps with explicit success criteria | "Just start coding" with no plan, or 20-step plans for 2-step problems |
| **Act** | Execute the next step (edit a file, run a command, call a tool) | Wrong tool, wrong args, side effects in the wrong place |
| **Verify** | Check the result against the success criteria | Skipping verification, or trusting model self-reports |
| **Reflect** | Update memory / context with what was learned | No reflection at all — every session re-learns the same lessons |

**The single most important principle**: every step should produce evidence the next step can check.

Strong (verifiable): "Add a test that fails on the current behavior, then make it pass."
Weak (unverifiable): "Make it work."

The loop is fractal. A single tool call has a tiny loop inside it. A whole sprint has a giant loop around it. You're managing nested loops at every level.

See [`frameworks/01-agent-loop.md`](frameworks/01-agent-loop.md) for the deep dive.

### 2. The four coding pitfalls (Karpathy)

This repo's original contribution. Four LLM-coding traps and the principles that defuse them:

1. **Think before coding** — Don't assume. Surface tradeoffs. Ask when unclear.
2. **Simplicity first** — Minimum code that solves the problem. No speculative abstractions.
3. **Surgical changes** — Touch only what you must. Don't drive-by refactor.
4. **Goal-driven execution** — Define success criteria. Loop until verified.

These are guardrails against the most common failure modes of strong LLMs. They are **not** optional — they belong in every project's rules. See [`frameworks/02-coding-pitfalls.md`](frameworks/02-coding-pitfalls.md).

### 3. The six harness components

An "agent harness" is the scaffolding around the model that turns it into a working teammate. Treat your setup as **six components** to design intentionally. Most failures are because one or more of these is missing or undisciplined.

| Component | What it does | Failure mode if neglected |
|---|---|---|
| **Reasoning** | Pick the model and effort level; let it think in interleaved chunks | Defaulting to one model+setting forever; under- or over-spending on hard problems |
| **Memory** | What the agent knows across turns and sessions | Re-learning everything every session; bloated context windows |
| **Context** | What's in the window _right now_ — tools, system prompts, files, history | Cache misses, irrelevant tokens, tool descriptions eating the budget |
| **Skills** | Packaged capabilities (a folder + a markdown file) the agent can load on demand | Everything in CLAUDE.md, nothing scales, agent forgets how to do things |
| **Orchestration** | How multiple loops/agents/sessions coordinate | One agent doing 50 things badly; no parallelism; humans babysitting |
| **Governance** | Verification, evals, safety, audit, permissions | Throughput up, quality down, nobody catches regressions |

See [`frameworks/03-harness-components.md`](frameworks/03-harness-components.md) and the per-component files in [`components/`](components/).

---

## What changed in 2026 (and why your old workflow is probably wrong)

Across the Code with Claude 2026 sessions, the same observations kept coming up. If you internalize these, you'll make better decisions about everything else.

### The bottleneck moved

**The slow part isn't writing code anymore. It's verification, review, coordination, and figuring out what to build.** (Fiona Fung, _Running an AI-native engineering org_)

If your processes were built when coding was the bottleneck — six-month roadmaps, design-doc-before-any-code, blanket code review, sequential planning rituals — they are quietly killing your throughput.

**Concrete shifts**:
- **In technical debates, code wins.** Generate two prototypes instead of arguing.
- **Reduce design docs.** Increase PR-as-prototype.
- **Shift verification _left_** — automation, hooks, evals — to catch issues before review.
- **Trust but verify** — let the agent do styling, lint, mechanical bug fixes, test writing. Reserve humans for taste, risk, security, legal.

### The context window is a fixed box

**Context windows are not getting bigger.** They've been roughly 200K–1M tokens for over a year and likely will stay there. Meanwhile models have gotten much better.

This means **context engineering is your highest-leverage skill**. You cannot solve agent problems by stuffing more into the prompt. You solve them by being surgical about what's in there.

Daisy Hollman's analogy: _"It's like trying to run npm on an Arduino."_

The KV cache adds a hard constraint: **stable content at the front, volatile content at the end.** Changing a token near the start of your prompt invalidates the cache for everything after it, which means 10× cost on the rest of the turn. So:

- Tool definitions and system prompts at the top (rarely change).
- Long-lived working memory in the middle.
- The current turn's task at the bottom.

See [`components/03-context.md`](components/03-context.md) for the full mechanics.

### Pick abstractions that scale

This is Daisy Hollman's central thesis. When you decide where to put something — MCP server, skill, hook, subagent, CLAUDE.md — the question isn't "where does it fit?" but **"what happens if I have 100,000 of them?"**

The hierarchy, cheapest to most expensive in context-window terms:

| Primitive | Token cost | Scales? | Best for |
|---|---|---|---|
| **Hooks** | Zero unless they fire | ✅ Yes | "Red squigglies" — corrections at the moment of a mistake |
| **Skills** | Description always loaded (~paragraph); body is pay-per-use | ⚠️ Mostly | Reusable capabilities, on-demand knowledge |
| **Subagents** | Description in parent prompt; body in separate context | ⚠️ Mostly | Parallelism, isolating context-heavy work |
| **MCP tools** | Name + description + schema always in prompt | ❌ No, without help | Cross-tool integrations, public APIs |
| **`CLAUDE.md` / system prompt** | Always loaded, always paid for | ❌ No | Truly universal project rules only |

**Rule of thumb**: if you already have a CLI, write a skill that uses it. Don't wrap it in MCP unless you're shipping to non-technical users. (Daisy)

### Tools that scale with intelligence > tools that compensate for it

Two flavors of tooling decisions:

- **Compensate for lack of intelligence**: hard-block the agent from doing X. _Doesn't scale with smarter models — they'll find legitimate reasons to do X._
- **Scale with intelligence**: nudge the agent with a hint or warning at the right moment. _Better models use the hint more effectively._

Daisy's example: don't hard-block edits to a generated file. _Hint_ the agent that it's a generated file. A smart model will respect that; a smarter model will respect it _and_ figure out the right place to make the change.

This applies everywhere. Permissions that block. Linters that fail builds. Rate limits. Style enforcers. **Prefer nudges over hard stops** wherever the cost of being wrong is low.

### Build for the next model

> "Things that didn't work months ago work now. Keep retrying use cases that previously failed whenever a new model ships." — multiple speakers, paraphrased

Hand-in-hand: **maintain evals so you can actually tell when the new model unlocks a previously-impossible use case.**

If you're optimizing your prompts and harness against today's model only, you'll bake in patches that fight tomorrow's model. (Margot Vanlar's _Prompting Playbook_ has the canonical example: a "never give the customer the wrong plan" patch that worked for an older model became a behavior bug on a smarter one that withheld information it actually knew.)

### Cheapest per successful outcome, not cheapest per token

> "The right model is the one that is cheapest per successful outcome." — Lucas, _Picking the right model_

Counterintuitive examples from the talks:
- Opus with high effort finished a coding task with **fewer total tokens** than Sonnet, because it took fewer turns.
- Haiku with the "advisor" pattern (calling Opus for the hard subset of cases) gets Opus-level intelligence at near-Haiku cost.
- A web search use case reduced input tokens by 77% and _accuracy went up 9%_ — because cleaning up tool output made Claude reason over less data.

The lesson: don't optimize a single dial. Run real evals, measure the outcome you actually care about, and pick the model+effort+context combination that minimizes cost per success.

### Measure outcomes, not activity

> "Acceptance rate is okay. Survival rate is better." — Mario Rodriguez, GitHub

If the AI generates code that gets accepted and then deleted next week, you didn't accomplish anything. Track:

- **Outcome metrics**: bug-fix survival rate, PRs merged that don't get reverted, customer task completion rate.
- Not just: tokens used, files changed, PRs opened.

---

## The six components, in plain language

You'll go deep on each in [`components/`](components/). Here's the orientation.

### Reasoning

Pick the right **model** (size class: Haiku / Sonnet / Opus, or equivalent) and the right **effort level**. Both can change per task.

Three test-time-compute mechanisms (Alexander Bricken, _The thinking lever_):
1. **Thinking** — internal scratchpad before output.
2. **Tool calls** — interface with the outside world.
3. **Text output** — what you actually see.

**Adaptive thinking** lets the model interleave thinking with tool calls, deciding _when_ to think. This beats always-on or always-off thinking because the model can match cognitive effort to task complexity.

**Heuristics**:
- _Latency-sensitive, low intelligence_ (classification, summarization, extraction): smaller model, low effort.
- _Default for real work_: extra-high effort. (Claude Code's default.)
- _Hardest tasks_: max effort, but watch for diminishing returns.
- _Cost-bound but intelligence-bound_: smaller model + advisor pattern (junior calls senior for the hard 5%).

See [`components/01-reasoning.md`](components/01-reasoning.md).

### Memory

What persists across turns, sessions, and agents. Three layers:

1. **Working memory** — short-lived notes for the current task, often a file the agent edits as it goes.
2. **Project memory** — semi-persistent: project conventions, decisions, gotchas. The `CLAUDE.md` family, or `.cursor/rules/`, or `AGENTS.md`.
3. **Knowledge base** — large, durable, possibly shared across agents. Runbooks, internal docs, past session learnings.

**File-system-based memory** has won. Frontier models are good at managing files with `bash` and `grep`. Don't over-engineer this — don't build a vector DB you don't need.

**Dreaming** (Anthropic's term, but the concept generalizes) = a batch out-of-band process that reads recent session transcripts and updates memory. Run it nightly or after major work. It catches patterns no single agent notices.

The four memory files in this template:
- `PROJECT.md` — what this project is, scope, constraints.
- `DECISIONS.md` — ADR-lite: decisions made and why.
- `PROGRESS.md` — recent state, what's in flight.
- `GOTCHAS.md` — non-obvious things, things-that-bit-us, lessons.

See [`components/02-memory.md`](components/02-memory.md) and [`memory/`](memory/).

### Context

The thing in the model's attention right now. The single biggest lever, and the most under-invested.

**Three principles**:

1. **Cache discipline.** Aim for 80–90% prompt cache hit rate (95%+ at scale). The fastest way to drop your costs by an order of magnitude is to stop invalidating the cache. Concretely: no UUIDs / timestamps / random IDs in the prefix. Tool definitions stable. Messages append-only.
2. **Shape your tool output.** Markdown over JSON. Strip fields you don't need. Convert timestamps to human-readable day-of-week. (Real example: 66% token reduction _and_ accuracy went up.)
3. **Don't pay for what you don't use.** Every token in the prompt is paid for every turn whether the model uses it or not. Apply zero-overhead-abstraction thinking from systems programming.

See [`components/03-context.md`](components/03-context.md) and [`rules/05-context-engineering.md`](rules/05-context-engineering.md).

### Skills

A **skill** is a folder containing a `SKILL.md` (with a one-line description in frontmatter) plus any supporting files. The model decides when to load the full skill body based on the description.

Why skills beat alternatives:
- vs. CLAUDE.md: pay only when triggered, not on every turn.
- vs. MCP: lighter weight, no auth/lifecycle, can shell out to existing CLIs.
- vs. subagents: faster, no separate context to manage.

**Design rules**:
- One skill = one purpose.
- Description must be informative enough for the model to know when to load it (~one paragraph). Test by asking: "Given the description alone, would I know to use this?"
- Body should be self-contained: scripts, references, examples.
- Avoid 50 micro-skills; avoid 1 mega-skill. Hierarchies are not yet well-supported; design for flat.

See [`components/04-skills.md`](components/04-skills.md) and [`skills/SKILL.md.template`](skills/SKILL.md.template).

### Orchestration

How multiple loops/agents/sessions coordinate. Several patterns:

- **Subagents** — sibling loops with isolated context, summoned by the parent for context-heavy subtasks (research, reading 50 files, parallel investigation).
- **Worktrees** — multiple checkouts of the same repo so you can run one agent per workstream without them stepping on each other. You become a "technical lead of multiple Claudes" (Daisy).
- **Routines / cron / `/loop`** — scheduled or event-triggered agents. "Coding agents shouldn't wait for you to press enter." (Maya, _Build a proactive agent workflow_) Examples: documentation drift, CI babysitting, daily summaries, CVE patches.
- **Generator → Evaluator → Repair** — three small prompts instead of one big one. (Margot's _Prompting Playbook_ shows this dramatically outperforming a single mega-prompt for schedule-generation.)
- **Advisor / critic / rubber-duck patterns** (GitHub) — junior model executes, senior model advises on hard cases or critiques plans.

**Async + parallel + context switching** is the high-leverage workflow. (Daisy is explicit about this.) Tips:
- Rename sessions, change colors, use status views.
- Use mobile/remote check-ins to sanity-check overnight runs.
- Cap the number of concurrent agents at your honest context-switching capacity, not your max.

See [`components/05-orchestration.md`](components/05-orchestration.md).

### Governance

Everything that keeps quality high as throughput goes up.

Concrete pieces:

- **Evals.** Not "smoke test, ship it." A real eval suite with: control cases (always pass), edge cases (the model has failed here before), and boundary cases (escalate / refuse / hand off). Run on every prompt change, every model change. **Small + well-designed beats big + sloppy** every time.
- **Online evals beat offline evals.** Offline sets the baseline; online (real traffic, A/B tested) tells you the truth.
- **Hooks as red squigglies.** Fire only when relevant, inject context only when needed. Best primitive for safety-critical nudges.
- **Auto mode + adversarial checks.** Anthropic's auto mode runs a classifier + adversarial agent before letting actions through. Costs ~30–40% more in tokens, but unlocks overnight and parallel work without permission prompts. Worth it for many workflows.
- **Permissioning by trust level.** Different agents get different scopes. Read-only memory for some, read-write for others.
- **Read the transcripts.** (Lucas insists.) Headline metrics lie. The transcript shows you what actually happened, including the time Claude solved a benchmark by reading git history for the answer.

See [`components/06-governance.md`](components/06-governance.md).

---

## How to actually use this directory

There's a temptation to copy everything everywhere. **Don't.** Start small:

### Day 0: minimum viable setup

For a fresh project:

1. Copy [`frameworks/02-coding-pitfalls.md`](frameworks/02-coding-pitfalls.md) into your project's `CLAUDE.md` / `AGENTS.md` / `.cursor/rules/` (whichever your tool reads).
2. Copy [`memory/PROJECT.md.template`](memory/PROJECT.md.template) into your project root as `PROJECT.md` and fill in the first 3 sections.
3. Pick one adapter file ([`adapters/claude-code.md`](adapters/claude-code.md), [`adapters/cursor.md`](adapters/cursor.md), or [`adapters/generic-agent.md`](adapters/generic-agent.md)) and follow the install steps for your primary tool.

That's it. You're done for the day.

### Day 1–7: add as you feel the pain

You'll feel one of these pains within the first week:

| Pain | Add |
|---|---|
| "Why is it making the same mistake every session?" | `memory/GOTCHAS.md`. Update it whenever a mistake recurs. |
| "I keep re-explaining the same workflow" | A skill in `skills/`. |
| "It's stomping on adjacent code" | The surgical-changes rule. |
| "It's hallucinating capabilities" | The eval skeleton from [`rules/06-verification-and-evals.md`](rules/06-verification-and-evals.md). |
| "Costs are climbing" | [`rules/05-context-engineering.md`](rules/05-context-engineering.md) — start with cache hit rate. |
| "It's too cautious / over-asking" | The simplicity-first rule, and re-read the talk-section on patches from older models. |
| "Two agents stepped on each other" | Worktrees + read [`components/05-orchestration.md`](components/05-orchestration.md). |

### Day 30+: refactor your harness

Re-read this guide. Now you have intuition for which sections matter for _your_ work. Audit:

- Are you measuring _outcomes_ or activity?
- What's your cache hit rate?
- What workflow have you _not_ Claudified yet that you should?
- What old process is still serving you, and which one quietly stopped working?

This is the audit Fiona Fung recommends. Do it quarterly.

---

## Anti-patterns (things I've seen people do)

- **One mega-prompt for everything.** Split into generator/evaluator/repair, or into skills, or into subagents.
- **Stuffing CLAUDE.md with every convention.** You pay for those tokens every turn. Move stable conventions into a skill the model loads when relevant.
- **Building an MCP server when you already have a CLI.** Write a skill instead.
- **Optimizing for prompts that work on today's model only.** You're baking in patches the next model will fight.
- **Hard-blocking instead of nudging.** Compensates for current weakness, fights future strength.
- **Trusting the headline benchmark number.** Read three random transcripts. You'll learn more.
- **Single-agent serial workflow.** You have parallelism available. Use worktrees, subagents, and routines.
- **No evals.** You cannot tune what you cannot measure.
- **All evals offline.** Offline catches regressions, online catches reality.
- **Never re-evaluating processes.** "Processes rarely kill themselves." (Fiona) — schedule an audit.

---

## What's NOT in this guide

A few intentional omissions:

- **Specific model recommendations.** They change every few months. Use the heuristics in [`components/01-reasoning.md`](components/01-reasoning.md) and re-pick when a new model lands.
- **RAG architectures.** A whole field, and most uses can be replaced by good context engineering + skills + file-system memory. Add RAG only when you have a clear retrieval problem.
- **Specific MCP server recommendations.** Same reason as models — ecosystem is moving. The general guidance ("don't wrap your own CLI in MCP") is stable.
- **Fine-tuning.** Mostly not worth it for product teams. Frontier models churn faster than you can fine-tune, and fine-tuning on niche data tends to increase hallucinations (per 2025 papers; Daisy mentions this).

---

## Where this came from

Every claim in this guide traces to one of:

- Andrej Karpathy's posts on LLM coding (linked in `CLAUDE.md`).
- The 36 transcripts in [`../transcripts/code-with-claude-2026/`](../transcripts/code-with-claude-2026/). Specific attribution lives in the per-component files.
- My own opinionated synthesis where the sources disagree or are silent. Marked as such.

If you disagree with a specific claim, the per-component file usually has the original quote. Argue with the source, not with me.

---

## What to read next

- **Just starting?** → [`README.md`](README.md) (this directory's map).
- **Want the conceptual scaffolding?** → [`frameworks/`](frameworks/).
- **Want concrete rules to copy into your project?** → [`rules/`](rules/).
- **Want to wire this up for a specific tool?** → [`adapters/`](adapters/).
- **Working on a specific component?** → [`components/`](components/) — six files, pick one.
