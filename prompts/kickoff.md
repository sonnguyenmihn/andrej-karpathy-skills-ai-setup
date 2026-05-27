# Prompt: Project Kickoff

> Paste this at the start of a new project or a new major workstream. Replace `{...}` placeholders.

---

I want to start working on **{project name}**. Before we write any code, help me build the foundation.

## Step 1: Understand the goal

Ask me up to 5 clarifying questions about:
- What this project is and isn't.
- The target user and their primary job-to-be-done.
- Hard constraints (deadline, stack, deployment target).
- What "done" looks like for the first milestone.

Wait for my answers before continuing.

## Step 2: Draft `PROJECT.md`

Based on my answers, draft a `PROJECT.md` with these sections:
- **What** — one paragraph, no jargon.
- **Why** — the user problem this solves.
- **Scope** — explicit in-scope and out-of-scope lists.
- **Constraints** — non-negotiable tech / time / quality bars.
- **Success criteria for milestone 1** — verifiable.

Show me the draft. Wait for my edits before continuing.

## Step 3: Draft the first plan

Once `PROJECT.md` is approved, propose a plan for milestone 1:
- Numbered steps.
- Each step has a verifiable check.
- Total ≤ 8 steps. If more, the milestone is too big.

Surface assumptions and open questions. Don't pick silently.

## Step 4: Confirm before code

Wait for me to approve the plan before writing any code.

---

**Rules of engagement** for this project (apply throughout):

- Think before coding. Surface tradeoffs. Ask when unclear.
- Simplicity first. No speculative abstractions.
- Surgical changes. Touch only what you must.
- Goal-driven. Every step has a verifiable check.
- When you finish a step, update `PROGRESS.md` with what changed and what's next.
- When you hit a non-obvious gotcha, add it to `GOTCHAS.md`.
- When you make a non-trivial decision, log it in `DECISIONS.md` with the alternatives considered.
