# Component: Reasoning

> **Pick the right model. Pick the right effort. Let it interleave thinking with action.**

## The three test-time-compute mechanisms

Every reasoning decision is a budget across three things (Alexander Bricken, _The thinking lever_):

1. **Thinking** — internal scratchpad before output. No external effect; pure cognition.
2. **Tool calls** — interface with the outside world. Reading files, running commands, searching.
3. **Text output** — what the user sees.

Older "reasoning models" thought once at the start, then tool-called. **Adaptive thinking** (current frontier) lets the model interleave thinking blocks between tool calls, deciding _when_ to think.

> "Adaptive thinking isn't a model router… it's telling Claude, 'you have this thinking tool.' Now Claude doesn't have to think at all if it doesn't need to." — Alexander Bricken

## Picking the model tier

Three classes, roughly:

| Tier | Use for |
|---|---|
| **Small** (Haiku, equivalent) | Classification, summarization, extraction, latency-bound interactions |
| **Mid** (Sonnet, equivalent) | Most coding tasks, agentic workflows with medium reasoning needs |
| **Large** (Opus, equivalent) | Hard reasoning, long-horizon agentic tasks, anything requiring planning over many steps |

**Counterintuitive**: bigger model + low effort often beats smaller model + max effort, _and_ uses fewer tokens overall because it takes fewer turns. (Real example from "Picking the right model": Opus high-effort finished a task in fewer tokens than Sonnet high-effort.)

**Heuristic**: if the task requires _any_ intelligence at all, start with the bigger model at lower effort. Tune downward if cost/latency demands it.

## Picking the effort level

Five levels (vendor terminology varies; these are the conceptual buckets):

| Effort | When to use | Notes |
|---|---|---|
| **Low** | Latency-sensitive, low-intelligence tasks: extraction, classification, summarization | Sometimes produces _better_ creative results by constraining reasoning |
| **Medium** | Balanced general purpose | Reasonable default for non-coding interactive use |
| **High** | Any coding task that needs real reasoning | "If you need any intelligence, land here or higher" |
| **Extra-high** | Default for Claude Code & Claude.ai | Best Pareto trade-off |
| **Max** | Hardest tasks only — known to be intelligence-bound | Watch for diminishing returns; can use 2× tokens for 5% gain |

**When in doubt**: extra-high. (This is Anthropic's own product default.)

## Don't toggle thinking on/off — let the model decide

The pre-adaptive pattern of "thinking on for hard tasks, off for easy ones" is an anti-pattern. Toggling thinking off doesn't reduce effort — it removes a capability.

> "When you turned extended thinking off, you just remove that capability from Claude. Now that's an un-ideal outcome." — Alexander Bricken

Analogy: you don't tell teammates "don't use your inner monologue for this." You give them the task and trust them to think (or not) as needed.

## Patterns that shift the cost curve

These don't just move you along the cost/quality curve — they shift the whole curve:

### Advisor (small calls big)

- _Executor_ is the small/fast model handling the bulk of cases.
- _Advisor_ is the large/slow model called only on the hard subset.
- Result: near-Advisor quality at near-Executor cost.

GitHub's Copilot uses this with Haiku-as-executor + Opus-as-advisor for the cases Haiku can't handle. (Brad Adams, Mario Rodriguez, _Caching, harnesses, and advisors_.)

### Rubber-duck / critic

Insert a critique step at high-leverage moments:
1. **After drafting a plan** — catch flawed plans before you spend tokens executing them.
2. **After a complex implementation** — pre-code-review.
3. **After writing tests but before running them** — especially when CI is slow.

GitHub calls this "Rubber Duck"; the pattern is general. Most ROI from inserting the critique at the _plan_ step.

### Generator → Evaluator → Repair (orchestrated loops)

For tasks with many constraints (scheduling, validation, complex outputs):
1. Generator produces a candidate.
2. Evaluator (separate prompt) reports violations.
3. Repair (third prompt) makes targeted fixes.

Margot Vanlar showed this beating both "Sonnet with better prompt" and "Opus with adaptive thinking" on a constraint-heavy scheduling task — _fewer tokens, lower latency_.

### Prompt caching

Most leverage you can get on cost without changing anything else. See [`components/03-context.md`](03-context.md).

## How to measure: build a small eval first

> "A small, well-designed eval will teach you so much more about which model to use than any public benchmark ever could." — Lucas, _Picking the right model_

Minimum eval for picking models:

- 5–20 tasks representative of your production workload.
- Each task has _inputs_, a _success criterion_, and ideally _expected steps_ (LLM-as-judge can check both outcome and process).
- Three case types: control (always passes), edge cases (model has failed here before), boundary (refuse / escalate / hand off).

Run the eval across:
- Multiple model tiers.
- Multiple effort levels.
- With and without thinking.

Plot accuracy × cost × latency. The right model is the cheapest per **successful outcome**, not the cheapest per token.

Common gotchas Lucas calls out:
- **Mistaking noise for signal** — run each task multiple times; check variance.
- **Infra failures** — separate API/tool failures from model failures when scoring.
- **Silent saturation** — your eval set must keep growing with new failure modes from production.

## Reading guide

- Daisy Hollman, _Beyond the basics with Claude Code_ (London) — context for why reasoning interacts with the harness.
- Lucas, _Picking the right model_ (London) — canonical talk on this topic.
- Alexander Bricken, _The thinking lever_ (London) — adaptive thinking & effort levels.
- Brad Adams + Mario Rodriguez, _Caching, harnesses, and advisors_ (SF) — advisor pattern.

→ [`../transcripts/code-with-claude-2026/london/10-picking-the-right-model.md`](../../transcripts/code-with-claude-2026/london/10-picking-the-right-model.md)
→ [`../transcripts/code-with-claude-2026/london/15-the-thinking-lever.md`](../../transcripts/code-with-claude-2026/london/15-the-thinking-lever.md)
