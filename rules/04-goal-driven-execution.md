# Rule: Goal-Driven Execution

> Drop-in rule. Copy into your project's `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/`, or equivalent.

**Define success criteria. Loop until verified.**

Transform imperative tasks into verifiable goals:

| Instead of… | Use… |
|---|---|
| "Add validation" | "Write tests for invalid inputs, then make them pass." |
| "Fix the bug" | "Write a test that reproduces it, then make it pass." |
| "Refactor X" | "Ensure tests pass before and after." |
| "Improve performance" | "Reduce p99 latency to under 300ms; benchmark in `bench/api.ts` reports it." |

For multi-step tasks, state a plan with verifiable checks per step:

```
1. <step> → verify: <check>
2. <step> → verify: <check>
3. <step> → verify: <check>
```

**Karpathy's framing**: _"LLMs are exceptionally good at looping until they meet specific goals. Don't tell it what to do; give it success criteria and watch it go."_

Strong success criteria let the agent loop independently. Weak criteria ("make it work") force you back into the loop constantly.

— Distilled from Karpathy on LLM coding pitfalls.
