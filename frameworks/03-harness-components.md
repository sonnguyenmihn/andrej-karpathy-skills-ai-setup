# Framework: The Six Harness Components

> **In one sentence**: An "agent harness" is the scaffolding around the model that turns it into a working teammate. Design it as **six components**: reasoning, memory, context, skills, orchestration, governance.

If your agent setup feels janky, one of these six is almost certainly under-engineered.

## Why this framing

The model is a fixed asset — you don't tune its weights, you don't retrain it. Everything that distinguishes a useful AI workflow from a frustrating one is _around_ the model. That scaffolding is what we mean by "harness."

The six components are functional: they describe _what_ the scaffolding does. Different tools (Claude Code, Cursor, custom agents) implement them differently. The components are stable; the implementations rotate.

## The six components

### 1. Reasoning

**What**: Pick the model class (Haiku / Sonnet / Opus, or equivalent) and the effort level (low / medium / high / max). Let the model interleave thinking with tool calls.

**Why it matters**: Test-time compute is a primary lever. A bigger model with low effort often beats a smaller model with max effort. Effort levels above "high" show diminishing returns on most tasks.

**Operational defaults**:
- Default to extra-high effort (Claude Code default).
- Use low effort for latency-bound tasks (classification, summarization, extraction).
- Use max effort only when you _know_ the task is intelligence-bound.

→ [`components/01-reasoning.md`](../components/01-reasoning.md)

### 2. Memory

**What**: Persistent state the agent reads and writes across turns and sessions.

**Why it matters**: Without memory, every session re-learns the same lessons. With unstructured memory, the context window drowns.

**Three layers** to keep separate:
- _Working memory_ — current task notes (often a file the agent edits as it goes).
- _Project memory_ — conventions, decisions, gotchas (`CLAUDE.md` family, `.cursor/rules/`, `AGENTS.md`).
- _Knowledge base_ — large, durable, possibly shared across agents (runbooks, past learnings).

File-system memory has won. Don't build a vector DB you don't need.

→ [`components/02-memory.md`](../components/02-memory.md)

### 3. Context

**What**: The tokens in the model's attention window _right now_ — system prompt, tool definitions, prior turns, current task.

**Why it matters**: Context windows are fixed (~200K–1M tokens), not growing. Every token in the prompt is paid for on every turn. The KV cache imposes a "stable prefix, volatile suffix" constraint or you'll pay 10× on cache misses.

**Three principles**:
1. **Cache discipline** — aim for 80–90%+ cache hit rate.
2. **Shape your tool output** — markdown over JSON, strip fields, dedupe.
3. **Don't pay for what you don't use** — zero-overhead-abstraction thinking from systems programming.

→ [`components/03-context.md`](../components/03-context.md)

### 4. Skills

**What**: Packaged capabilities (a folder + a `SKILL.md`) the agent loads on demand.

**Why it matters**: Skills are a middle abstraction. Cheaper than putting everything in `CLAUDE.md` (which pays the cost every turn). Lighter than MCP (no auth, no lifecycle). Better than no abstraction (the agent can't find what it doesn't know about).

**Design rules**:
- One skill = one purpose.
- Frontmatter description must be informative enough to trigger correctly (~one paragraph).
- Body is self-contained (scripts, references, examples).
- Avoid 50 micro-skills and 1 mega-skill. Design for flat.

→ [`components/04-skills.md`](../components/04-skills.md)

### 5. Orchestration

**What**: How multiple loops/agents/sessions coordinate — sequentially, in parallel, on schedule, or reactively.

**Why it matters**: One agent doing everything is the slow path. Real productivity comes from running several loops in parallel, isolating context-heavy work in subagents, and putting recurring work on a schedule (routines).

**Patterns**:
- _Subagents_ — sibling loops with isolated context for context-heavy subtasks.
- _Worktrees_ — multiple repo checkouts so agents don't step on each other.
- _Routines / scheduled agents_ — proactive work without waiting for human input.
- _Generator → evaluator → repair_ — three small prompts beating one mega-prompt.
- _Advisor / critic / rubber-duck_ — junior model executes, senior critiques at key moments.

→ [`components/05-orchestration.md`](../components/05-orchestration.md)

### 6. Governance

**What**: Everything that keeps quality high as throughput goes up — evals, hooks, permissions, audits.

**Why it matters**: Throughput went up. Verification didn't, by default. If you don't actively shift verification left, quality silently degrades.

**Pieces**:
- _Evals_ — small, well-designed test suites run on every prompt/model change.
- _Online evals_ — A/B testing in production. Offline sets baseline; online tells truth.
- _Hooks_ — "red squigglies" for agents. Fire only when relevant.
- _Auto-mode / classifier+adversary_ — unlocks overnight & parallel work without permission spam.
- _Permission scopes_ — different agents get different read/write access.

→ [`components/06-governance.md`](../components/06-governance.md)

## How the components map to the agent loop

| Loop phase | Components most involved |
|---|---|
| Perceive | Memory (read), Skills (load on demand), Context (what's loaded) |
| Think | Reasoning (effort level), Context (scratchpad space) |
| Plan | Reasoning, Skills (planning skills) |
| Act | Skills (capabilities), Orchestration (which agent runs the step) |
| Verify | Governance (evals, hooks, graders) |
| Reflect | Memory (write), Governance (audit log) |

If a phase is consistently weak, look at the components that support it.

## Common imbalances

I've seen each of these:

- **All reasoning, no memory.** Pays for max-effort thinking every turn, re-learns the same lessons every session.
- **All context, no skills.** `CLAUDE.md` is 10K tokens and counting. Cache hit rate dropping. Skills folder has 2 entries.
- **All skills, no governance.** 200 skills, no evals, nobody knows which ones still work.
- **All orchestration, no memory.** Five agents in parallel, none can find what the others learned.
- **All governance, no reasoning.** Evals are pristine, but the model is wrong tier for the task.

Use the [`README.md`](../README.md) map to identify which 1–2 components you're weakest on and start there.

---

**Source attribution**: The six-component framing is the user's contribution. I've populated each component from the Code with Claude 2026 transcripts. Specific quotes attributed per component file.
