# Rule: Simplicity First

> Drop-in rule. Copy into your project's `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/`, or equivalent.

**Minimum code that solves the problem. Nothing speculative.**

- **No features beyond what was asked.**
- **No abstractions for single-use code.**
- **No "flexibility" or "configurability"** that wasn't requested.
- **No error handling for impossible scenarios.**
- **If you write 200 lines and it could be 50, rewrite it.**

Ask yourself: _"Would a senior engineer call this overcomplicated?"_ If yes, simplify.

The most common LLM coding failure is **over-engineering on under-specified tasks**. Factories, dependency injection, config flags, "extensible architecture" — resist all of it unless the user asked for it.

When in doubt: ship the minimum. You can add complexity later when you have a concrete reason to.

— Distilled from Karpathy on LLM coding pitfalls.
