# Prompt: Debug a Problem

> Paste when something's broken. Replace `{...}` placeholders.

---

**Symptom**: {what you're seeing — error message, wrong output, unexpected behavior}

**Expected**: {what should be happening instead}

**What I've already tried**: {list any debugging you've done so far, OR "nothing yet"}

---

Debug this by following these steps. Do them in order; don't skip.

## 1. Reproduce

Before forming any hypothesis, write the smallest test or command that reliably reproduces the bug. Show me what you tried and the output. If you can't reproduce, ask me for more information — don't guess.

## 2. Form hypotheses

List 2–3 distinct hypotheses for the root cause. Be specific:
- Bad: "There's a bug in the auth code."
- Good: "The token expiry check uses local time but the JWT was minted in UTC, so cross-zone clocks cause false-positive expiry."

## 3. Cheapest hypothesis first

Rank your hypotheses by how cheap they are to test. Test the cheapest one first.

## 4. Verify with evidence

For each hypothesis you test:
- State what you'd expect to see if the hypothesis is right.
- State what you'd see if it's wrong.
- Run the test. Report what you actually saw.

Don't say "this confirms my theory" unless the evidence specifically distinguishes your theory from the alternatives.

## 5. Fix and verify the fix

When you've found the root cause:
- Propose the smallest fix that addresses it (not a refactor).
- Add a regression test that fails without the fix.
- Apply the fix. Confirm the test passes.

## 6. Reflect

Update `GOTCHAS.md` with:
- The symptom (so future-you can find this entry by searching).
- The root cause (one or two sentences).
- The fix.

---

**Rules of engagement**:
- Don't change unrelated code while debugging.
- Don't suppress errors as a "fix."
- If two hypotheses are equally cheap to test, test both.
- If after 3 hypotheses you haven't reproduced, stop and ask.
