---
name: mw-split
description: Split a grilled feature summary into small local tickets in tickets/. Use whenever the user wants to break a feature, grill summary or scope into tickets or tasks, or says "split this into tickets", "break it down", "make tasks from this". Runs after mw-grill in the Minimalistic Workflow.
model: sonnet
disable-model-invocation: true
---

# mw-split

Turn an agreed scope into tickets that can each be planned, built and reviewed on their own.

You always propose the split yourself, in both modes. The user only confirms it.

## Input

The grill summary in `.mw/grill/<slug>.md`. If there is none, ask the user to run `mw-grill` first or to paste the agreed scope. Do not invent scope.

Use the mode recorded in the summary. If none is recorded, ask once: "think or ship?" Copy the mode into every ticket; later steps read it from there.

## How to split

- One ticket = one reviewable PR. If a ticket would touch many unrelated areas, split it.
- Prefer vertical slices (a thin working feature end to end) over horizontal layers ("all entities", then "all endpoints"), unless a shared foundation must land first.
- Put foundations (migrations, shared types) in their own early ticket.
- Every ticket must have acceptance criteria that can be checked by a test or a clear manual step.
- Size: S (under half a day), M (about a day), L (more). Split every L.
- **Covers** points to the grill decisions and edge cases this ticket implements, by name. Never restate or paraphrase them: the grill summary stays the single source, and `mw-plan` reads it directly.
- Every in-scope grill decision and edge case appears in at least one ticket's Covers. List any that do not when you show the split.

## Ticket format

Number tickets after the highest existing number in `tickets/`. File: `tickets/NNN-<slug>.md`.

```markdown
# NNN: <title>

**Feature:** <grill slug>
**Mode:** think | ship
**Size:** S | M
**Depends on:** NNN, NNN | none

## Goal

<one or two sentences>

## Covers

- Decision: <grill decision, named as in the summary>
- Edge case: <grill edge case, named as in the summary>

## Acceptance criteria

- [ ] ...

## How to verify

<optional: only when the ticket has user-facing behaviour. Numbered manual steps to check the change in the UI or API.>

## Out of scope
```

Show the list (number, title, size, dependencies) and ask for confirmation before writing the files.

Next step: `mw-plan` on the first ticket with no open dependencies.
