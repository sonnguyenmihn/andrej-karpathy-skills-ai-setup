# Component: Orchestration

> **How multiple loops/agents/sessions coordinate. The shift from "I prompt, it answers" to "I direct a team."**

## The mental model shift

> "Your job as a software engineer at this point is to make little clones of yourself so you can scale up your abilities and scale up your work across many agents." — Daisy Hollman

This is the orientation. You're not pair-programming with one Claude. You're a tech lead managing several. Some run synchronously (you're watching), some run asynchronously (overnight, on schedule, on events).

Orchestration is the engineering of that team.

## Pattern 1: Subagents

A subagent is a child loop with its own context, summoned by the parent for a specific task. Useful for:

- **Context-heavy subtasks** — reading 50 files, exploring a foreign codebase, doing research. The parent gets back a summary, not 50 file contents.
- **Specialized tasks** — a "test-writer" subagent with a focused system prompt and instructions.
- **Parallel investigation** — three subagents investigate three hypotheses simultaneously.

Cost shape: the subagent's system prompt and content live in a **separate context window**. The parent pays only for the tool call invoking the subagent and the result returned. (Daisy)

Design rules (from skill design, applied here):
- One subagent = one purpose.
- Description in the parent prompt must clearly say when to use it.
- Result returned to parent should be _summarized_, not raw.

## Pattern 2: Worktrees

Multiple checkouts of the same repo, each in its own directory, each with its own agent.

```
~/proj/
├── main/             ← human's terminal, planning
├── wt-feature-a/     ← agent 1
├── wt-feature-b/     ← agent 2
└── wt-bugfix-c/      ← agent 3
```

`git worktree` handles this cleanly. Daisy uses this as her default workflow: long-lived agents that own specific directories, each tracking upstream main, each with persistent identity.

Why it works:
- Agents don't step on each other.
- You don't pay tokens to context-switch the agents — each one keeps its own session.
- You context-switch _yourself_ between them, which is fine if the sessions are well-labeled.

Tips:
- **Rename sessions** and **change colors**. Daisy: _"Color actually does trigger memory pretty efficiently."_
- Keep each worktree's task focused. One workstream per worktree.
- A separate "planning" / "coordinating" agent in the main directory can orchestrate the others.

## Pattern 3: Routines (scheduled & event-triggered agents)

> "Coding agents shouldn't wait for you to press enter to get started." — Maya, _Build a proactive agent workflow_

A routine is an agent that runs:
- On a schedule (cron).
- On an event (GitHub PR opened, Slack message, webhook).
- On user-defined triggers.

Examples that pay off immediately:
- **Documentation drift** — nightly: "Read recent merged PRs. Update docs that drift from the code."
- **CVE / dependency patches** — weekly: "Check for security advisories on dependencies. Open PRs for safe patches."
- **CI babysitting** — on PR push: "If CI fails on style or trivial test issues, fix and re-push."
- **Customer feedback summarization** — daily: "Read #customer-feedback. Summarize. Open issues for actionable items."
- **Sprint retrospectives** — end-of-sprint: "Read PR titles, commit messages, incidents. Draft a retro."

The "/loop" command in Claude Code is the most general form: run this prompt every N minutes, the agent can turn it off when no longer relevant.

## Pattern 4: Generator → Evaluator → Repair

Instead of one mega-prompt trying to do everything, split into three:

1. **Generator** — produces a candidate (a plan, a schedule, an output, a draft).
2. **Evaluator** — separate prompt, separate (possibly smaller) model, checks the candidate against constraints and reports specific violations.
3. **Repair** — receives violations, makes targeted fixes.

Loop generator → evaluator → repair until evaluator passes (with a max-iteration cap).

Margot Vanlar (_The prompting playbook_) showed this beating both "Sonnet with better prompt" and "Opus with adaptive thinking" on a constraint-heavy scheduling task — **fewer tokens, lower latency, all test cases pass**.

Why it works:
- Each prompt has a single clear objective. (See [`02-coding-pitfalls.md`](../frameworks/02-coding-pitfalls.md#4-goal-driven-execution).)
- Soft constraints can be added to evaluator at runtime without retraining the generator.
- Failures are localized — you can see whether the generator or the evaluator is wrong.

## Pattern 5: Advisor / Critic / Rubber-duck

Inject a senior model's review at key moments without using it for the bulk of work.

| Pattern | Junior | Senior | When senior fires |
|---|---|---|---|
| **Advisor** | Executor | Advisor | On hard subtasks the executor can't crack |
| **Critic** | Generator | Critic | After plan; after complex implementation; after tests are written |
| **Rubber duck** | Implementer | Critic | At explicit checkpoints (GitHub's pattern) |

Mario Rodriguez's three high-leverage critique moments:
1. After drafting a plan (highest ROI).
2. After complex implementation, pre-code-review.
3. After writing tests, before running them (especially when CI is slow).

## Pattern 6: Auto-mode + adversarial check

Anthropic's "auto mode" lets agents act without permission prompts by running:
- A classifier on each proposed tool call.
- An adversarial agent that tries to find why this call might be unsafe.

Cost: ~30–40% more tokens. Benefit: overnight & parallel work without babysitting.

> "This is what makes loop usable. This is what makes agent teams usable. This is what makes overnight work usable." — Daisy

Use auto mode when:
- You're running multiple agents in parallel and can't approve each action.
- You're running overnight / on schedule.
- The agent is in a sandboxed environment where mistakes are recoverable.

Don't use it (without extra guardrails) when:
- The agent has write access to production systems with no rollback.
- The agent has access to secrets / billing / privileged operations.

## Async + parallel + context switching

The combination is the highest-leverage workflow. Daisy:

> "Asynchrony, where you can walk away from the computer, let it work for a while and come back. And parallelism, where you really want to be doing multiple of these things at a time. The combination means you just really have to get good at context switching."

Tooling to support this:
- **Mobile remote control** of agents — 30-second check-in after dinner.
- **Agent dashboard views** — see which agents are running, blocked, done.
- **Session naming and coloring** — fast context switching.
- **Persistent identity for long-lived agents** — agent N owns directory X.

## A practical orchestration setup

For a single developer with multiple workstreams:

```
~/proj/
├── main/                              ← you, planning + coordination
├── wt-feature-a/      [Agent A]       ← currently working on feature A
├── wt-feature-b/      [Agent B]       ← currently working on feature B
├── wt-docs/           [Routine: docs] ← nightly doc-drift fixer
└── wt-deps/           [Routine: deps] ← weekly dep patcher
```

Plus:
- A subagent pattern for "go read 30 files in this monorepo and tell me what depends on X."
- A critic pattern at every PR-creation moment.
- Auto mode on all routines.
- Each agent has a distinct color/name.

For a team: same patterns, plus permission scopes on shared memory and clear ownership of each routine.

## Anti-patterns

- **Many agents, one shared memory store, no concurrency control.** They clobber each other.
- **No identity / labeling on parallel agents.** You can't tell which is which after 2 hours.
- **Single mega-prompt for everything.** Split into generator/evaluator/repair or into smaller agents.
- **Auto mode without sandboxing.** "Auto mode meets prod database." Not great.
- **Subagent for trivial tasks.** Overhead exceeds benefit. Subagents earn their keep on context-heavy work.

## Reading guide

- Daisy Hollman, _Beyond the basics with Claude Code_ (London) — worktrees, async/parallel, agent teams, auto mode.
- Maya, _Build a proactive agent workflow with Claude Code_ (London) — routines specifically.
- Margot Vanlar, _The prompting playbook_ (London) — generator/evaluator/repair pattern.
- Brad Adams + Mario Rodriguez, _Caching, harnesses, and advisors_ (SF) — advisor & rubber-duck patterns.

→ [`../transcripts/code-with-claude-2026/london/02-beyond-the-basics-with-claude-code.md`](../../transcripts/code-with-claude-2026/london/02-beyond-the-basics-with-claude-code.md)
→ [`../transcripts/code-with-claude-2026/london/19-build-a-proactive-agent-workflow-with-claude-code.md`](../../transcripts/code-with-claude-2026/london/19-build-a-proactive-agent-workflow-with-claude-code.md)
