# Framework: The Four Coding Pitfalls (Karpathy)

> **In one sentence**: Frontier LLMs reliably fall into four traps when coding. These four principles defuse them.

This is the original contribution of this repo (`CLAUDE.md`). Restated here for completeness; if you've already internalized them, skip to [`03-harness-components.md`](03-harness-components.md).

## The traps (Karpathy's diagnosis)

> "The models make wrong assumptions on your behalf and just run along with them without checking. They don't manage their confusion, don't seek clarifications, don't surface inconsistencies, don't present tradeoffs, don't push back when they should."
>
> "They really like to overcomplicate code and APIs, bloat abstractions, don't clean up dead code… implement a bloated construction over 1000 lines when 100 would do."
>
> "They still sometimes change/remove comments and code they don't sufficiently understand as side effects, even if orthogonal to the task."
>
> — Andrej Karpathy

Four traps, four principles:

| Trap | Principle |
|---|---|
| Silent assumptions; no pushback | **Think before coding** |
| Bloat & speculative abstraction | **Simplicity first** |
| Side-effect changes; "improving" adjacent code | **Surgical changes** |
| Weak success criteria → constant clarification | **Goal-driven execution** |

## 1. Think before coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

The Code with Claude 2026 talks reinforce this from multiple angles:

- Margot Vanlar: _"If you can't tell guidelines from policy from data, the model can't either."_ Apply structure (XML tags, clear sections) to the prompt before troubleshooting failures.
- Lucas (Picking the Right Model): _"Read your transcripts."_ Most "why did the model do X?" questions resolve when you actually see what the model saw.

## 2. Simplicity first

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If 200 lines could be 50, rewrite it.

The test: _Would a senior engineer call this overcomplicated?_

This is where models fight you the hardest. They _love_ adding factory patterns, dependency injection, config flags, and "future-proofing." Resist.

## 3. Surgical changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports / variables / functions that _your_ changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: _Every changed line should trace directly to the user's request._

## 4. Goal-driven execution

**Define success criteria. Loop until verified.**

Transform imperative tasks into verifiable goals:

| Instead of… | Use… |
|---|---|
| "Add validation" | "Write tests for invalid inputs, then make them pass." |
| "Fix the bug" | "Write a test that reproduces the bug, then make it pass." |
| "Refactor X" | "Ensure tests pass before and after." |
| "Improve performance" | "Reduce p99 from 800ms to under 300ms; benchmark in `bench/`." |

Karpathy's pithy version: _"Don't tell it what to do; give it success criteria and watch it go."_

For multi-step tasks, state a brief plan with a verify-check on each step. See [`../prompts/plan.md`](../prompts/plan.md).

## Tradeoffs

These principles bias toward **caution over speed**. For trivial tasks (typo fixes, obvious one-liners), use judgment. The goal is reducing costly mistakes on non-trivial work, not slowing down simple tasks.

## How to know it's working

- Fewer unnecessary changes in diffs — only requested changes appear.
- Fewer rewrites due to overcomplication — code is simple the first time.
- Clarifying questions come _before_ implementation, not after mistakes.
- Clean, minimal PRs — no drive-by refactoring or "improvements."

---

**Source attribution**: Karpathy's diagnosis and principles. See `CLAUDE.md` at the repo root for the canonical short version.
