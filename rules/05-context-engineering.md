# Rule: Context Engineering

> Drop-in rule. Copy into your project's `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/`, or equivalent.

**The context window is a fixed box. Don't pay for what you don't use.**

Context windows are not growing meaningfully (200K–1M tokens is the long-term shape). Models are. Every token in the prompt is paid for on every turn. Your job is to use that fixed budget well.

## Three principles

### 1. Cache discipline (aim for 80–90%+ hit rate)

The KV cache makes the prefix cheap and the suffix expensive. Cached tokens cost ~10× less than uncached.

- **No UUIDs, timestamps, or random IDs** in the system prompt or near the top.
- **Stable tool definitions** — don't dynamically reorder or swap them.
- **Treat the messages array as append-only.** Don't edit prior messages.
- **Measure cache hit rate** — your API returns the metric. Plot it. Hill-climb on it.

If you're hitting < 80%, something is invalidating the cache. Find it.

### 2. Shape your tool output

Tool responses are usually the largest source of waste in agent context.

- **Markdown over JSON** when the consumer is the model. Tables, lists, plain prose.
- **Strip fields the model doesn't need.** That metadata blob? Probably not necessary.
- **Pre-compute derived values.** Day-of-week from a date, totals from rows. Don't make the model do mental math.
- **Dedupe.** Multiple tool calls returning overlapping items? Dedupe before passing back.

Real example: shaping a sports-data tool's output reduced tokens 66% _and_ accuracy went up 9%.

### 3. Choose abstractions that scale

When deciding where to put a customization, ask: _"what happens if I have 100,000 of these?"_

Cheapest to most expensive in token terms:

| Abstraction | Token cost per turn |
|---|---|
| Hook | 0 unless it fires |
| Skill body | 0 unless loaded |
| Skill description | ~paragraph |
| Subagent description | ~paragraph in parent |
| MCP tool (full def) | Name + description + schema, always |
| `CLAUDE.md` content | Full content, always |

Prefer the cheaper abstraction when both work. **Don't wrap your own CLI in MCP** — write a skill that uses the CLI.

## Self-check

- [ ] Cache hit rate measured? Above 80%?
- [ ] Any dynamic content in the system prompt? (UUIDs, timestamps, random IDs)
- [ ] Tool responses larger than they need to be?
- [ ] Is `CLAUDE.md` carrying things that should be in skills?
- [ ] Any MCP servers wrapping a CLI you already have?

— Distilled from Daisy Hollman (_Beyond the basics with Claude Code_), Mario Rodriguez (_Caching, harnesses, and advisors_), Lucas (_Picking the right model_).
