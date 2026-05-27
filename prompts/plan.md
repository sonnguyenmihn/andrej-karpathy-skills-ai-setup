# Prompt: Plan a Task

> Paste before starting any non-trivial task. Replace `{...}` placeholders.

---

I want to: **{one-line goal}**

Before writing any code, produce a plan in exactly this shape:

```
## Task
<restate the goal in one line>

## Assumptions
- <each assumption you're making, named explicitly>

## Plan
1. <step> → verify: <how I'll check this step worked>
2. <step> → verify: <how>
3. <step> → verify: <how>

## Open questions
- <anything unclear or blocking>

## Out of scope
- <related things you considered but are NOT doing>
```

Rules for the plan:

- **Steps ≤ 7.** If you need more, your steps are too small or the task is too big — split or rescope.
- **Each verify is concrete.** "Tests pass" is okay if there are tests. "Looks right" is not.
- **Assumptions are surfaced**, not buried. If the task could be interpreted multiple ways, list the interpretations.
- **Open questions block work.** If any open question makes you guess, stop and ask me instead.

If you can't fit the work into this shape — for example, the task is exploratory and steps depend on what you find — say so, and propose a smaller first step that _can_ be planned cleanly.

Show me the plan. Wait for my approval before writing code.
