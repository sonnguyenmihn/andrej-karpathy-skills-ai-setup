# Framework: The Agent Loop

> **In one sentence**: Every useful agent runs a fractal loop of _perceive → think → plan → act → verify → reflect_. Most failures are missing or weak phases.

## The phases

```
   ┌─────────────────────────────────────────────────────┐
   │                                                     │
   ▼                                                     │
 perceive ──▶ think ──▶ plan ──▶ act ──▶ verify ──▶ reflect
                  │                          │            │
                  └────── ask / clarify ◀────┘            │
                                                          │
                                  (memory updates) ◀──────┘
```

| Phase | What happens | Good signals | Bad signals |
|---|---|---|---|
| **Perceive** | Load relevant files, tool results, prior context | Reads ≤ 5 _right_ files | Reads 50 files / reads no files / re-reads same file every turn |
| **Think** | Reason about state, options, risks (scratchpad / `<thinking>` block) | Names assumptions, surfaces uncertainty | Jumps straight to action |
| **Plan** | Choose a sequence of steps with success criteria per step | "Write failing test → make it pass → refactor" | "Implement the thing" |
| **Act** | Execute one step | Atomic, reversible, observable | Multi-edit megabatches that can't be reviewed |
| **Verify** | Check the result against the criteria | Runs tests, reads back the diff, inspects output | "Looks good!" |
| **Reflect** | Update memory / notes / GOTCHAS with what was learned | Specific lessons logged | Discards everything, next session re-learns |

## Why "loop", not "pipeline"

The loop is _re-entrant_ at every level:

- A whole project is a giant loop (a quarter-long iteration).
- A sprint is a smaller loop.
- A single task is a smaller one still.
- A single tool call has a tiny loop inside it (the model decides, calls, reads the result, decides again).

You're managing nested loops. Most agentic failures come from missing the verify/reflect phases at one or more levels.

## The single most important property: verifiability

Every phase output should be evidence the next phase can check.

| Verifiable | Not verifiable |
|---|---|
| "Add a test that fails on the current behavior, then make it pass." | "Make it work." |
| "Refactor X — tests must pass before and after." | "Refactor X." |
| "Add input validation; the test suite under `tests/validation/` must pass." | "Add validation." |
| "Reduce p99 latency to under 300ms; benchmark in `bench/api.ts` reports it." | "Make it faster." |

Karpathy's framing: _"LLMs are exceptionally good at looping until they meet specific goals. Don't tell them what to do; give them success criteria and watch them go."_

## What goes wrong, by phase

**Perceive fails when**: the agent doesn't have access to what you have. Slack threads, design docs, CI logs, dashboards. (Daisy Hollman: _"If Claude can't get to the things you can get to, it can't do your job with you."_)

**Think fails when**: the model is over-confident on under-defined tasks. Defuse this with the [think-before-coding rule](../rules/01-think-before-coding.md): state assumptions, surface alternatives, ask when unclear.

**Plan fails when**: there is no plan, or the plan has no success criteria. The simplest fix: require a numbered plan with a verify-check on each step before any code is written.

**Act fails when**: changes are too coarse to review. Force smaller commits / smaller edits / smaller PRs. "PR-as-prototype" beats "PR-as-epic."

**Verify fails when**: humans are the verification mechanism. Shift verification _left_ via tests, linters, hooks, and grader agents. (Fiona Fung: _"Shifting verification left is the only way to maintain quality as throughput increases."_)

**Reflect fails when**: nothing is written down. Every session re-learns the same lessons. Fix: a `GOTCHAS.md`, and/or a "dreaming" process that periodically scans recent sessions and consolidates.

## Operationalizing the loop in your setup

1. **Reasoning** lives in _think_ — pick the right model & effort for the task.
2. **Skills** and **MCP tools** support _perceive_ and _act_.
3. **Hooks** patrol the boundary between _act_ and _verify_.
4. **Memory** is what _reflect_ writes to and _perceive_ reads from.
5. **Orchestration** is how multiple loops nest and run in parallel.
6. **Governance** is the evals + audits that grade whether the loop is working.

See [`03-harness-components.md`](03-harness-components.md) for the component mapping.

## A working template for tasks

For any non-trivial task, the agent should produce this before any code:

```
## Task
<one-line goal>

## Assumptions
- ...
- ...

## Plan
1. <step> → verify: <how>
2. <step> → verify: <how>
3. <step> → verify: <how>

## Open questions
- ...
```

If the assumptions or open questions block forward progress, the agent should _stop and ask_, not guess. See [`../prompts/plan.md`](../prompts/plan.md) for the prompt template.

---

**Source attribution**: This framing is Karpathy's. The phase definitions are my synthesis. The "verifiability" insistence is straight from Karpathy's posts (quoted in this repo's main `README.md`).
