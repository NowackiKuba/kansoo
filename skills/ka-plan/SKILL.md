---
name: ka-plan
description: Write an implementation plan for a ticket and stop for human approval before any code is written. Use whenever the user wants to plan a ticket, task or change, says "plan this", "make a plan", "how should we do this", or is about to start implementing a ticket from tickets/. Runs before ka-build in Kansoo.
model: opus
disable-model-invocation: true
---

# ka-plan

A plan is the contract `ka-build` implements and `ka-review` checks against. Fixing a plan costs minutes; fixing code costs hours.

The plan covers only **how** to build the ticket. The **what** and **why** (goal, scope, decisions, edge cases, risks) were settled in the brief and are fixed input. Refer to them; never restate them or ask the user to write them again.

## Input

- The ticket in `tickets/NNN-*.md`, or the task the user described.
- The brief in `.ka/briefs/<slug>.md`, where `<slug>` is the ticket's `**Feature:**`. The ticket's `## Covers` names which of its decisions and edge cases this ticket implements. If there is none (e.g. a task without a grill), the ticket is the only source of decisions; list anything it leaves undecided under New findings.

## Core principle (think mode)

The user produces the first version of every decision (scope, plan, hypothesis, review findings). You critique it, ask questions and execute. Never present your own full solution first. Reveal your own version only after the user's version exists, or when the user explicitly asks for it.

## Mode

Use the mode recorded in the ticket. If none is recorded and the user did not say, propose a mode with a one-sentence reason and let them confirm or override it:

- **think** if the change touches any of the "Sensitive areas" list in `~/minimalistic-workflow/README.md`; or touches paths you have not changed in the last ~3 months (`git log --author="$(git config user.email)" --since=3.months --oneline -- <affected paths>` prints nothing); or involves a design decision that is hard to reverse.
- **ship** otherwise.

Example: `Proposed mode: think — touches migrations and queue retries. OK?` Accept a one-word reply: `ok` keeps the proposal, `think` or `ship` overrides it.

What the modes mean here:

- **think:** the user writes the plan, you review it.
- **ship:** you write the plan, the user approves.

Record the mode in the plan.

## Steps (think mode)

1. Read the ticket and the brief (see Input).
2. Ask the user to write only **Approach** and **Shape** (see Plan format). Tell them it must be detailed enough that implementation needs no design decisions, and that grill decisions need not be repeated. They may mark any section `[human]` to implement it themselves. Wait for it.
3. Read `CLAUDE.md`, the relevant docs from its map, and the code the plan touches: callers, tests, similar existing features.
4. Review the user's plan against the code, docs and grill decisions: missing callers or other existing code the change must reach, existing patterns they should reuse, grill edge cases the shape does not handle, and vague parts that would force a design decision during build. Raise each as a question, with `file:line` where it helps. Do not rewrite the plan.
5. Draft **Existing code touched**, **Tests** and **New findings** yourself.
6. When the user has answered, write the plan, in their decisions, to `.ka/plans/NNN-<slug>.md`. Show it and **stop**. Do not edit any source file until the user approves.
7. After approval, offer (do not force) to show how you would have planned it, for comparison.

## Steps (ship mode)

1. Read the ticket and the brief (see Input).
2. Read `CLAUDE.md`, then the docs from its map that are relevant to this ticket.
3. Read the code you will change and the code around it: callers, tests, similar existing features. Reuse existing patterns.
4. Write the plan to `.ka/plans/NNN-<slug>.md` in the format below.
5. Show the plan and **stop**. Ask for approval. Do not edit any source file until the user approves.

## Plan format

A heading or list item marked `[human]` is implemented by the user; `ka-build` skips it.

- **Approach:** how, in 2–4 sentences. No why: that lives in the brief.
- **Shape:** the code-level decisions build would otherwise have to make: new modules/files and where they live, key types and signatures, which existing pattern each part follows, and the order of work.
- **Existing code touched:** only existing files that are modified or deleted (callers, shared modules, migrations, config). New files belong in Shape. Docs are never listed: `ka-build` does not touch them, and `ka-review` reports the ones the change made stale.
- **Tests:** one line per acceptance criterion and per edge case in the ticket's `## Covers`, so nothing decided in the grill goes untested.
- **New findings:** only what reading the code revealed that the grill and ticket did not decide: an unhandled case, a conflict with the architecture, or **code that contradicts a grill decision** (the grill may be out of date). Each is a question for the user, never a silent choice. Omit the section if empty.

```markdown
# Plan: NNN <title>

**Mode:** think | ship
**Brief:** .ka/briefs/<slug>.md | none

## Approach

## Shape

## Existing code touched

- path/to/file.ts — modify / delete: what

## Tests

- <test case> — <AC or grill edge case it proves>

## New findings

- <finding> — <question for the user>
```

## After feedback

Update the plan file with the user's corrections and show only what changed. Count each round of corrections.

## Metrics

On approval: `{"step":"plan","ticket":"NNN","date":"<ISO date>","mode":"think|ship","revisions":<rounds of corrections>}`
