---
name: ka-build
description: Implement an approved plan from .ka/plans/, or the code findings of a ka-review report from .ka/reviews/, following the project's architecture and conventions, and loop on tests, typecheck and lint until they pass. Use whenever the user approves a plan, says "go", "build it", "implement it", "approved, do it", asks to implement a ticket that has an approved plan, or to fix review findings. Runs after ka-plan in Kansoo.
model: sonnet
disable-model-invocation: true
---

# ka-build

The approved plan is the source of truth. Architecture and conventions are the standard the code must meet.

## Core principle (think mode)

The user produces the first version of every decision (scope, plan, hypothesis, review findings). You critique it, ask questions and execute. Never present your own full solution first. Reveal your own version only after the user's version exists, or when the user explicitly asks for it.

## Before coding

1. Read the approved plan in `.ka/plans/NNN-*.md` (given a `ka-review` report instead, follow Review fixes below). If there is no approved plan, stop and suggest `ka-plan`. Use the mode recorded in the plan; if none is recorded, ask once: "think or ship?"
2. Read the ticket, the brief named in the plan's `**Brief:**` line (the plan refers to its decisions without restating them), `CLAUDE.md`, and the architecture and conventions docs.
3. Find the test, typecheck and lint commands in `CLAUDE.md`. If they are missing, read `package.json` (or equivalent) and ask the user to confirm them. Run every check through `ka-run <test|typecheck|lint> -- <command>` (e.g. `ka-run test -- pnpm test`): it prints `ok (<kind>)` on success and only the useful part of the output on failure.

## Implement

- **Never touch docs.** Do not create, edit or delete `README.md`, `CLAUDE.md`, anything under `docs/`, or any file in the `CLAUDE.md` docs map — not even when the plan lists it or a doc is now wrong. `ka-review` reports stale docs and applies the fixes. Note the stale doc in the report instead.
- Create new files as the plan's Shape and the project's conventions dictate. Modify or delete no existing file outside the plan's `Existing code touched` list unless required for the plan to work.
- Write the tests listed in the plan along with the code.
- Match existing patterns in the codebase over your own preferences.
- Skip every part of the plan marked `[human]`. Leave a `TODO [human]: <what, plan section>` comment where that code belongs.
- **Think mode:** implement only what the plan specifies. If the plan leaves a design decision open, stop and ask the user to decide. Never fill the gap yourself.

## Review fixes

When given a `ka-review` file (`.ka/reviews/NNN.md`) instead of a plan:

- If the file already has an `## Applied` section, stop and suggest `ka-review`: those fixes are waiting for re-review.
- Read it, then the plan, brief and ticket it reviewed (if they exist), and do step 3 of Before coding. Use the mode recorded in the plan; if none, ask once.
- Fix every finding in the file, in order, nits included. The user already chose them in review: do not ask which to apply and do not fix anything not in the file.
- Each finding's `file:line`, what to change and evidence are the spec. When a finding allows more than one real fix: **think mode** ask the user which; **ship mode** pick one and name it in the report.
- If a fix needs a design change the plan does not cover, treat it as a wrong plan (see below).
- All other rules apply: never touch docs, verification loop, report. Skip the comprehension check unless the fixes changed behaviour.
- When the checks are green, append `## Applied` to the review file: one line per finding, `N. file:line` of its fix (several if it took more than one place). `ka-review` re-reviews only those files and deletes the file when clean.

## Verification loop

After implementing, run tests, typecheck and lint through `ka-run`.

- All green → done.
- Red → read the actual error (if the compact output is not enough to diagnose it, rerun only that single failing test with full output), fix the cause (never skip, delete or weaken a test, and never add ignore comments to make it pass), run again.
- Maximum 3 fix attempts. After the third failure, stop and report: what fails, what you tried, your best hypothesis.
- If a check is red only because a `[human]` part is not written yet, do not work around it. Report which failures belong to it.

## When the plan is wrong

If the plan cannot work as written, or you need a change it does not cover, **stop and ask**. Do not silently deviate: an unannounced deviation is exactly what review would otherwise have to catch.

- **Ship mode:** describe the problem and propose an amendment to the plan.
- **Think mode:** describe the problem and ask the user how to amend the plan. Do not propose the amendment unless they ask.

## Comprehension check (think mode)

After the checks are green, ask the user 2–3 questions that make them trace what the code in this diff does: given this input or state, what happens at `file:line`? Each question must be answerable by reading the diff. Do not give the answers. If an answer is wrong or vague, point to the exact `file:line` and let them try again.

Never ask:
- **Why a decision was made** when the plan or ticket made it. The user made that decision; quizzing them on it tests nothing.
- **About a bug you suspect.** If you think something can break, report it as a finding with `file:line`. Do not hide it in a quiz.
- **About code outside the diff**, or about hypotheticals the code does not handle.

## Report

When green, report: files changed, tests added, any approved deviation, a summary of the verification output, every `TODO [human]` left for the user (file, plan section, what to write), and any doc the change made stale (for `ka-review`'s docs check).

Never stage or commit. Leave every change uncommitted in the working tree: `ka-review` reviews uncommitted changes (`git diff HEAD`) by default, and committing is `ka-commit`'s job after review.

Next step: `ka-review` in a fresh session.

## Metrics

`{"step":"build","ticket":"NNN","date":"<ISO date>","mode":"think|ship","attempts":<verification runs>,"deviations":<count>,"result":"green|stopped"}`
