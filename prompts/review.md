# Prompt: Code Review (Critic Pattern)

> Paste before merging non-trivial code. Useful as a routine on every PR. Replace `{...}` placeholders.

---

Review the following code change. Be a critic, not a cheerleader.

**Change**: {diff, PR link, or pasted code}

**Context**: {what this is supposed to do, link to relevant `PROJECT.md` or issue}

---

## Review in this order

### 1. Does it do what's claimed?

- Trace the change against the stated goal.
- Are there cases the change is _supposed_ to handle that it misses?
- Are there cases the change handles that it shouldn't (scope creep)?

### 2. Correctness

- Edge cases: empty inputs, nulls, off-by-one, concurrent access, timezones.
- Error paths: what happens when each external call fails?
- Type / runtime: any unchecked assumptions about types or shapes?

### 3. Safety

- Authentication / authorization changes — is the new code reachable by unauthorized callers?
- Secrets / PII handling.
- Destructive operations — could this delete or overwrite data without recovery?
- New dependencies — are they vetted?

### 4. Surgical-ness

- Are there changes unrelated to the stated goal? Flag them.
- Are there formatting / style changes that bloat the diff?
- Could the same effect be achieved with less code?

### 5. Tests

- Is there a test that would fail without this change? If no, this is a red flag.
- Are tests _real_ — do they exercise the actual behavior, or just call the function and assert it returns _something_?
- Mock usage: any places where mocks hide real failure modes?

### 6. Maintainability

- Will the team understand this in 6 months?
- Any subtle invariants the code depends on but doesn't document?
- Names: do variables and functions name the thing they actually represent?

---

## Output format

For each category above, give one of:

- **OK** — nothing flagged.
- **Concern**: <specific concern, ideally with line reference>
- **Blocker**: <something I think prevents merging>

End with one of:

- **Recommend merging.** (Briefly say why.)
- **Recommend changes.** (List the required changes.)
- **Recommend rejection.** (Explain why this approach is wrong.)

Be direct. If I'm being lazy, say so. If the approach is bad, propose a better one.
