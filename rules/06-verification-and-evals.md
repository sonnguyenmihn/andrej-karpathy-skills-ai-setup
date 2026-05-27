# Rule: Verification and Evals

> Drop-in rule. Copy into your project's `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/`, or equivalent.

**You cannot tune what you cannot measure. Shift verification left.**

## You need evals before you change prompts or models

A small, well-designed eval beats a huge sloppy one. Start with 5–20 cases:

- **Control cases** — should always pass. Sanity check.
- **Edge cases** — specific known failure modes from production.
- **Boundary cases** — when should the agent refuse / escalate / hand off?

Each case has: input, success criterion (programmatic or LLM-as-judge), optionally expected steps.

**Run the eval on every prompt change. Run it on every model change.** No exceptions.

## Don't optimize the wrong metric

> "Acceptance rate is okay. Survival rate is actually a better metric." — Mario Rodriguez

Track outcomes, not activity:

- Not "lines generated." Yes "PRs merged that don't get reverted."
- Not "tickets handled." Yes "tickets resolved without re-open."

## Online evals tell the truth; offline evals set the baseline

Offline (synthetic): fast, cheap, approximate. Run on every change.

Online (real traffic, A/B): slow, expensive, real. Run before promoting a new default. Weekly reports.

If you only have one, choose online. Or rather: choose to build both, in that order.

## Three eval gotchas (Lucas)

1. **Noise vs. signal** — run each case multiple times. High variance means the case is poorly defined.
2. **Infra failures != model failures** — separate them when scoring. Don't blame the model when the API timed out.
3. **Silent saturation** — keep adding new production failure modes to the eval. Otherwise your eval becomes a museum.

## Shift verification left

Throughput is up. By default, verification didn't scale with it. Compensate actively:

- **Hooks** as "red squigglies" — catch issues at the moment, not at PR review.
- **Linters / formatters / type-checkers** wired into the agent's loop.
- **Test-after-edit** — run affected tests, inject failures back to the agent.
- **Auto-fix when safe** — let the agent fix style / lint / mechanical issues without asking.

## Read your transcripts

> "You really need to read your transcripts of what the agent or model is doing." — Lucas

Headline metrics lie. Every week, pick 3 random transcripts. Read them end to end. You will find:

- Agents solving benchmarks by reading prior trial outputs.
- Agents working around your safety patches in clever, unintended ways.
- Tool responses you didn't realize were terrible.

This is not optional governance. It's the single highest-ROI hour of your week.

— Distilled from Lucas (_Picking the right model_), Fiona Fung (_Running an AI-native engineering org_), Mario Rodriguez (_Caching, harnesses, and advisors_).
