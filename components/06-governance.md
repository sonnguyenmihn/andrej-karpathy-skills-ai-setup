# Component: Governance

> **Everything that keeps quality high as throughput goes up. Evals, hooks, permissions, audit, and the discipline to actually use them.**

## The throughput problem

When agents make you 5× faster at writing code, you don't automatically get 5× more reviewers. You don't automatically get 5× better tests. You don't automatically get 5× more vigilance on security.

The result, if you don't actively counter it: quality silently degrades.

> "Verification has been doubled down on. Shifting verification left is the only way to maintain quality as throughput increases." — Fiona Fung

Governance is what makes the throughput sustainable.

## Pillar 1: Evals

### Why evals

You cannot tune what you cannot measure. Every speaker at Code with Claude 2026 said this, in some form. The version I like best is Lucas's:

> "A small, well-designed eval will teach you so much more about which model to use than any public benchmark out there ever could."

### What an eval looks like

A small set of test cases (5–50 is fine to start, you'll grow it) where each case has:

- **Inputs** — the prompt or scenario the agent receives.
- **Success criterion** — programmatic (assertion) or LLM-as-judge.
- **Expected steps** (optional but valuable) — LLM-as-judge on the trace, not just the final answer.

Three case categories:

| Category | Purpose |
|---|---|
| **Control** | Should always pass. Sanity check. |
| **Edge cases** | Specific known failure modes from production. |
| **Boundary cases** | Refuse / escalate / hand off — checking what the agent _shouldn't_ do |

Lucas's gotchas:

1. **Mistake noise for signal.** Run each case multiple times; check variance. If you see big swings, the case is poorly defined.
2. **Infra failures.** Distinguish API/tool failures from model failures. Don't blame the model when your sandbox crashed.
3. **Silent saturation.** Add new failure modes from production back into the eval set. Otherwise your eval becomes a museum.

### Offline vs. online evals

Both matter, but they answer different questions:

| | Offline | Online |
|---|---|---|
| Speed | Fast | Slow (needs traffic) |
| Cost | Cheap | Real money |
| Truth | Approximation | Ground truth |
| When | Every prompt change, every model change | Before promoting a new default; ongoing |

Mario Rodriguez's rule: _"Offline gives you an indication, but it's not going to be 100% reality. You learn a lot more from your online evals after launch."_

Practical: A/B test with a small slice of real traffic. Weekly reporting. Track outcome metrics (see below).

### Measuring outcomes, not activity

> "Acceptance rate is okay. Survival rate is actually a better metric. If you ended up accepting it and then deleted after, that did not accomplish the outcome." — Mario Rodriguez

For coding agents:
- Bad: "Lines of AI-generated code merged."
- Better: "PRs merged that don't get reverted."
- Best: "Time-to-completion of the user's actual goal."

For customer-facing agents:
- Bad: "Tickets handled by the bot."
- Better: "Tickets resolved without re-opening."
- Best: "Customer satisfaction or net retention impact."

Pick the outcome that genuinely matters to your business, and track it.

## Pillar 2: Hooks ("red squigglies for agents")

A hook is a script that fires on agent events: before tool use, after tool use, on session start, etc. It returns text to inject into the context, or nothing.

> "Hooks are the only abstraction that don't consume context until they fire." — Daisy Hollman

The mental model is "the squiggly red line in your IDE":
- It catches mistakes _at the moment_, not at PR review.
- It's a nudge, not a hard block (where possible).
- It scales — adding a 100th hook doesn't cost more context unless the 100th hook fires.

Good hook examples:

| Hook | Purpose |
|---|---|
| Post-edit linter | Inject lint errors back to agent for self-correction |
| Pre-bash typo-checker | Catch `rm -rf` typos before they run |
| File-edit guard | Warn (don't block) on edits to generated files |
| Test-after-edit | Run affected tests after each edit, inject failures |
| Secret-scanner | Block (this one's worth blocking) commits with apparent secrets |

### Hooks: nudge vs. block

A pattern from Daisy's "tools that scale with intelligence":

- **Block** for safety-critical, never-acceptable actions: secret in code, push to main, drop-table-without-confirmation.
- **Nudge** for "usually wrong, but the model may know better": editing generated files, modifying lock files, changing existing tests instead of writing new ones.

Smarter models will respect nudges _and_ explain when they have a good reason to ignore them. Hard blocks fight smarter models.

## Pillar 3: Auto mode + adversarial checks

If you're running agents overnight or in parallel, you need _some_ way for them to act without prompting you for every operation. Anthropic's "auto mode" (and equivalents in other tools) does this via:

- A classifier reading each proposed action.
- An adversarial agent trying to find why the action might be unsafe.

Cost: ~30–40% more tokens. Benefit: unattended operation, agent teams, parallel work.

Use it. The cost is worth it for non-interactive workflows.

Combine with:
- Sandboxed environments (containers, VMs, ephemeral worktrees).
- Permission scopes (see below).
- Periodic mobile / dashboard check-ins.

## Pillar 4: Permission scopes

Not every agent needs write access to everything. Differentiate:

| Access level | Examples |
|---|---|
| **Read-only memory** | Shared knowledge bases, org-wide runbooks |
| **Read-write memory** | Per-team or per-task working memory |
| **Read-only filesystem** | Subagents doing research / exploration |
| **Read-write filesystem in sandbox** | Most coding agents |
| **Write access to repo** | Trusted agents under review |
| **Direct push to main / deploy to prod** | Specific, human-approved workflows only |

The principle: by default, an agent gets the least access it needs. Promote access by exception, not by default.

## Pillar 5: Audit & version history

Everything an agent does should be inspectable after the fact:

- **Memory updates** — version history with attribution (which agent, when, why).
- **Action transcripts** — store every session's tool calls and outputs.
- **Decisions** — when an agent makes a non-obvious choice, log the reasoning.

`git` is your friend for memory and code. For transcripts, your tool likely already captures them — make sure you're not deleting them.

When something goes wrong, the question "what did the agent see, decide, and do?" must be answerable. If it's not, your governance has a gap.

## Pillar 6: Read the transcripts

This is governance at the human level. From Lucas:

> "You like really need to read your transcripts of what the agent or model is doing at different points in your system."

Headline metrics lie. Concrete example: Lucas's team saw Claude scoring very well on a coding benchmark. The transcripts showed Claude was reading the git history of previous trial runs to find the answer. The metric was 100%; the actual progress was zero.

Routine: every week, pick 3 random transcripts. Read them end to end. You will be surprised.

## What good governance looks like

A team with strong governance:

- Has a small eval suite they run on every prompt change and every model swap.
- Reports outcome metrics, not activity metrics.
- Uses hooks for in-context safety nudges.
- Runs auto-mode with adversarial checks for parallel / overnight work.
- Has tiered permission scopes for different agents.
- Has version history on memory and easy access to transcripts.
- Has someone whose job (or part of whose job) is to read random transcripts and find weird behavior.

A team without strong governance:

- Notices regressions when customers complain.
- Has no idea what the actual cache hit rate is.
- Runs the same prompt on a new model and assumes it still works.
- Doesn't know which agents have access to what.

## Reading guide

- Fiona Fung, _Running an AI-native engineering org_ (SF + London) — verification shift-left, processes that stop serving you.
- Lucas, _Picking the right model_ (London) — evals, common gotchas, "read your transcripts."
- Mario Rodriguez + Brad Adams, _Caching, harnesses, and advisors_ (SF) — outcomes vs activity, online vs offline.
- Daisy Hollman, _Beyond the basics with Claude Code_ (London) — hooks, auto mode.

→ [`../transcripts/code-with-claude-2026/sf/13-running-an-ai-native-engineering-org.md`](../../transcripts/code-with-claude-2026/sf/13-running-an-ai-native-engineering-org.md)
→ [`../transcripts/code-with-claude-2026/london/10-picking-the-right-model.md`](../../transcripts/code-with-claude-2026/london/10-picking-the-right-model.md)
