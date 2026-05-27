# Rule: Think Before Coding

> Drop-in rule. Copy into your project's `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/`, or equivalent.

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- **State your assumptions explicitly.** If uncertain, ask.
- **If multiple interpretations exist, present them** — don't pick silently.
- **If a simpler approach exists, say so.** Push back when warranted.
- **If something is unclear, stop.** Name what's confusing. Ask.

For non-trivial tasks, surface a brief plan with assumptions before writing code:

```
## Task
<one-line goal>

## Assumptions
- ...

## Plan
1. <step> → verify: <how>
2. <step> → verify: <how>

## Open questions
- ...
```

If the open questions are blocking, _stop and ask_. Do not guess.

— Distilled from Karpathy on LLM coding pitfalls; reinforced by Margot Vanlar ("if you can't tell guidelines from policy from data, the model can't either") and Lucas ("read your transcripts").
