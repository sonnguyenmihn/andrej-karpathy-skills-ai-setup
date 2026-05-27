# Rule: Surgical Changes

> Drop-in rule. Copy into your project's `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/`, or equivalent.

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- **Don't "improve" adjacent code**, comments, or formatting.
- **Don't refactor things that aren't broken.**
- **Match existing style**, even if you'd do it differently.
- **If you notice unrelated dead code, mention it** — don't delete it.

When your changes create orphans:

- **Remove imports / variables / functions that _your_ changes made unused.**
- **Don't remove pre-existing dead code** unless asked.

The test: _every changed line should trace directly to the user's request._

If you find yourself making "while I'm in here" changes, stop. Either ask the user if they want those changes too, or leave them for a separate PR.

— Distilled from Karpathy on LLM coding pitfalls.
