# Prompt: Reflect (End-of-Session)

> Run at the end of a working session, or as a routine after each meaningful chunk of work.

---

We just finished working on: **{what you did this session}**

Update the project memory with what we learned. Specifically:

## 1. PROGRESS.md

Update `PROGRESS.md`:
- Mark completed items as done with a date.
- Add new in-flight items.
- Write a one-paragraph "where we are" summary at the top, replacing the previous one.

## 2. DECISIONS.md

For any non-trivial choice we made this session, append to `DECISIONS.md`:

```
## YYYY-MM-DD: <one-line decision>

**Context**: <why this decision was needed>
**Choice**: <what we decided>
**Alternatives considered**: <what we didn't pick, and why>
**Reversibility**: <easy / moderate / hard to undo later>
```

Skip trivia. A decision is worth logging if a future-you reading this would say "interesting, I might have done it differently."

## 3. GOTCHAS.md

If we hit any non-obvious problem and figured it out, append to `GOTCHAS.md`:

```
## <symptom keywords>
**Symptom**: <how you'd notice this>
**Cause**: <what was actually going on>
**Fix**: <what worked>
**Note**: <anything else worth remembering>
```

If we _didn't_ figure something out and gave up: log it as an open gotcha. Future-us will appreciate the breadcrumbs.

## 4. Look for skill candidates

Did we repeat a procedure this session that we've done before? If yes, suggest extracting it into a skill:

```
**Skill candidate**: <name>
**Description**: <when to use it>
**Body sketch**: <bullet list of what the skill would say>
```

Don't create the skill yet — just propose it. I'll decide.

## 5. Look for rule candidates

Did we run into a model failure mode that good upfront instructions would prevent? If yes, suggest a rule for the project's `CLAUDE.md` / `AGENTS.md`.

---

Show me the diffs for each file you propose changing. Wait for my approval before writing.
