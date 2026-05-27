# Rule: Prompt Hygiene

> Drop-in rule. Copy into your project's `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/`, or equivalent.

**Clean prompts are correctable prompts. Messy prompts overfit, then fail on the next model.**

## General hygiene

Apply these before troubleshooting any specific failure mode:

- **Structure with delimiters** (XML tags, markdown headings). Separate _role_, _policy_, _guidelines_, _tone_, _data_, _user input_.
- **Remove copy-paste artifacts.** Cookie banners, hero-image references, leftover HTML.
- **Group like with like.** All policy in one section; all output rules in another.
- **One topic per section.** Don't mix "tone" and "billing rules" in the same paragraph.

The heuristic (Margot Vanlar):

> "If you're reading a prompt and you can't tell guidelines from policy from data, most likely the model isn't able to either."

## Output contracts

When the output format matters:

- **Explicit schema.** XML tags, JSON schema, or markdown structure stated up front.
- **Stop sequences.** Wire your API call to stop at the closing tag.
- **Structured outputs** (if your provider supports them) — programmatic guarantee, no parsing surprises.

This matters most when downstream code parses the output. For conversational outputs it's less critical.

## Instructions don't add capability

If the model can't do the thing, telling it "it's critical that you do the thing correctly" doesn't help.

Real example: a prompt told the model "always calculate prorated bills correctly." The model still got the math wrong. The fix wasn't a stronger instruction — it was **giving the model a calculator tool**.

If the model is failing at math, give it a calculator. If it's failing at search, give it a search tool. **Instructions don't compensate for missing capability.**

## State both sides of tradeoffs

A patch that says "avoid X because X is expensive" will cause the model to over-fit on never doing X — even when X is the right call.

Bad:
> "Avoid escalating to a human as it costs $8."

Better:
> "Escalating to a human costs $8. Failing to escalate when the customer has a billing error costs a refund plus customer trust. Choose accordingly."

Smarter models reason over the tradeoffs. Don't take one side of the argument away from them.

## Patches age badly

A patch that fixed an old model's behavior often _causes_ new failures on a better model.

Real example: an instruction "never give the customer the wrong plan, point them at the URL instead" was a patch for an older model that confabulated plans. On a newer model that actually knew the plan, the instruction caused it to withhold information the customer should have received.

**Track _why_ you added each defensive instruction.** When you upgrade the model, audit the patches — most should be removable.

A simple convention: comment your prompts.

```
<!-- [2025-09] Patch for model X confabulating plans. Re-evaluate on model upgrade. -->
```

## Three approaches in increasing power (Margot)

When a use case is failing:

1. **Better prompt.** Cleanup, structure, output contract. Often gets you most of the way.
2. **Better model / more thinking effort.** Throw more reasoning at it.
3. **Agentic loop.** Generator → evaluator → repair. Often _cheaper_ than (2) for complex constraint problems.

Try in this order. Don't jump to (3) when (1) hasn't been tried.

— Distilled from Margot Vanlar (_The prompting playbook_).
