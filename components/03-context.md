# Component: Context

> **The tokens in attention right now. The highest-leverage, most under-invested component in most setups.**

## The fixed-box reality

Context windows are roughly 200K–1M tokens. They've been there for over a year. Models keep getting better; context windows are not getting bigger.

> "We're fundamentally at a limit, or what looks like a limit, of context window size. The only way to get more efficient is to get better at not paying for what you don't use." — Daisy Hollman, _Beyond the basics_

This puts **context engineering** at the center of any serious agent setup.

## The KV cache constraint

The KV cache makes context "engineering" not "configuration." Specifically:

- Tokens are processed left-to-right.
- The KV cache stores intermediate computation for a prefix of tokens.
- If you change a token at position N, the cache invalidates from position N to the end.
- Cached input tokens cost about **1/10** of uncached input tokens.

**Practical consequence**: a stable prefix is dramatically cheaper than a volatile one. Daisy's framing:

> "These tokens are cheap. These tokens are expensive. You're going to pay a whole lot for a lot of expensive tokens just to save some context window."

The right ordering:

```
┌─────────────────────────────┐
│ System prompt (stable)      │ ← cached, cheap
├─────────────────────────────┤
│ Tool definitions (stable)   │ ← cached, cheap
├─────────────────────────────┤
│ Conversation history        │ ← mostly cached
├─────────────────────────────┤
│ Current turn (volatile)     │ ← uncached, expensive
└─────────────────────────────┘
```

**Implications**:
- No UUIDs, timestamps, or random IDs in the system prompt or near the top. Mario Rodriguez (GitHub) told the story of UUIDs in their system prompt invalidating the entire cache. Don't be them.
- Don't dynamically reorder tool definitions. Keep them stable.
- Treat the messages array as **append-only**. Don't edit prior messages; that invalidates the cache.

## The three context-engineering principles

### 1. Cache discipline (aim for 80–90%+ hit rate)

> "Run above 94–96% [cache hit rate]. If we operate at 70%, that usually means we have a bug." — Mario Rodriguez, GitHub Copilot

That's at GitHub's scale. For most teams, 80–90% is the target. Concretely:

- **Measure it.** Your API responses include cache hit metrics. Plot them. Watch the trend.
- **No dynamic prefix.** Stable system prompt, stable tool definitions, stable everything until the volatile bit at the end.
- **Append-only.** Once a message is in the array, treat it as immutable.
- **Hill-climb.** When you find a cache miss, find what changed and fix it.

### 2. Shape your tool output

Real example from "Picking the right model": a sports-data tool returned JSON with full ISO timestamps, redundant fields, etc. After cleanup (markdown instead of JSON, simpler timestamps, day-of-week added, dedup):

- **66% reduction in tokens.**
- **9% increase in accuracy** — because the model reasoned over less noise.

How to shape:

- **Markdown > JSON** when the consumer is the model. JSON wastes tokens on `{}`, `""`, and field names. Use tables/lists.
- **Remove fields the model doesn't need.** That metadata blob your API returns? Strip it.
- **Pre-compute** derived values. Day-of-week from a date, totals from rows, etc. Don't make the model do mental math.
- **Dedupe.** If multiple tool calls might return the same items, dedupe before passing them back.

### 3. Don't pay for what you don't use

Borrow zero-overhead-abstraction thinking from C++. Every token in the prompt is paid for every turn — even if it never gets used.

This is the lens you use to evaluate plugin abstractions (Daisy):

| Abstraction | Token cost per turn |
|---|---|
| Hook | 0 unless it fires |
| Skill body | 0 unless loaded |
| Skill description | ~paragraph |
| Subagent description | ~paragraph in parent |
| MCP tool (full def) | Name + description + schema |
| CLAUDE.md content | Full content |

Prefer the cheaper abstractions when both work.

## Long context windows are not more expensive

A common myth, debunked by Mario:

> "Long context windows does not mean you're spending more. In fact, what you have to understand is how compaction is being done."

Compaction (summarizing earlier turns to fit) is what drives cost up:
- Each compaction produces ~4K output tokens.
- Output tokens are ~5× input tokens.
- Compaction also invalidates the cache for everything before it.

So: **stay within the natural context window when you can.** Compact less, cache more.

## Tool definitions that scale

If you have many MCP tools and they're all loaded into the system prompt, your context fills up before the model sees a single line of your code.

Strategies:

- **Tool search / lazy loading** (Anthropic's pattern): names in system prompt, full schemas loaded on demand via a search tool. Works for non-generic tools (Slack, internal APIs). Doesn't work for generic tools (Edit, Bash) — those need to be available always.
- **Don't wrap your own CLI in MCP.** Write a skill that uses the CLI. (Daisy's rule of thumb.)
- **Trim tool descriptions.** Aggressively.

## A checklist for context audits

When you suspect context engineering is your bottleneck:

- [ ] Measure cache hit rate. Is it < 80%?
- [ ] Are there UUIDs / timestamps / random IDs in the system prompt or tool definitions?
- [ ] How many tools are loaded? How many are used in a typical session?
- [ ] What's the largest tool response by token count? Can it be shaped?
- [ ] Is the messages array being mutated, or appended only?
- [ ] What's in `CLAUDE.md` that could move to a skill?
- [ ] Are there scripts shelled out via `bash` that could replace MCP servers?

## Reading guide

- Daisy Hollman, _Beyond the basics with Claude Code_ (London) — the canonical "context is a fixed box" talk.
- Mario Rodriguez + Brad Adams, _Caching, harnesses, and advisors_ (SF) — practical cache discipline at GitHub scale.
- Lucas, _Picking the right model_ (London) — tool output shaping with concrete numbers.

→ [`../transcripts/code-with-claude-2026/london/02-beyond-the-basics-with-claude-code.md`](../../transcripts/code-with-claude-2026/london/02-beyond-the-basics-with-claude-code.md)
→ [`../transcripts/code-with-claude-2026/sf/05-caching-harnesses-and-advisors-building-on-claude-at-github-scale.md`](../../transcripts/code-with-claude-2026/sf/05-caching-harnesses-and-advisors-building-on-claude-at-github-scale.md)
