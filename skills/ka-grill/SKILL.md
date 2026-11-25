---
name: ka-grill
description: Grill a feature idea or requirement until nothing is ambiguous, grounded in the project's docs and code. Use whenever the user wants to discuss, clarify, scope, stress-test or grill a feature, change or idea before building it, or says things like "let's grill this", "let's scope this feature", "let's talk this through before we start". First step of the Kansoo feature flow.
model: opus
disable-model-invocation: true
---

# ka-grill

Turn a rough idea into a clear, agreed scope before any ticket or code exists. Questions are cheap now and expensive after implementation.

## Core principle (think mode)

The user produces the first version of every decision (scope, plan, hypothesis, review findings). You critique it, ask questions and execute. Never present your own full solution first. Reveal your own version only after the user's version exists, or when the user explicitly asks for it.

## 0. Mode

If the user did not say, propose a mode with a one-sentence reason and let them confirm or override it:

- **think** if the change touches any of the "Sensitive areas" list in `~/kansoo/README.md`; or touches paths you have not changed in the last ~3 months (`git log --author="$(git config user.email)" --since=3.months --oneline -- <affected paths>` prints nothing); or involves a design decision that is hard to reverse.
- **ship** otherwise.

Example: `Proposed mode: think — touches migrations and queue retries. OK?` Accept a one-word reply: `ok` keeps the proposal, `think` or `ship` overrides it.

What the modes mean here:

- **think:** the user writes first; the think-mode rules below apply.
- **ship:** you drive the questions and draft the brief; the user approves.

Record the mode in the brief so later steps follow it.

## 1. The user writes first (think mode)

Ask the user to write their own understanding of the feature:

- **Goal:** who is it for, what problem, how will we know it works
- **Scope:** what is in
- **Out of scope**
- **Edge cases**
- **Risks**

Wait for it. Do not suggest content for any of these points before the user's text exists.

In ship mode, skip this step.

## 2. Read first, ask second

Before the first question:

1. Read `CLAUDE.md` and its docs map. Read every doc relevant to the idea (at minimum architecture and conventions if they exist), plus any ADR in `docs/adr/` that touches the same area and `docs/GLOSSARY.md` if it exists. Use the glossary's terms in your questions and the brief.
2. Search the codebase for related modules, entities, endpoints and existing solutions.
3. Tell the user in 2–4 lines what you found: relevant docs, existing code that already does part of this, and any conflict with the architecture.

Do not ask anything the docs or code already answer. Asking it wastes the user's time and signals you did not read.

## 3. Grill

Ask 1–3 questions at a time, most important first. Cover, as relevant:

- **Goal:** who is this for, what problem, how will we know it works
- **Scope:** what is explicitly out of scope
- **Behaviour:** happy path, edge cases, errors, empty states, limits
- **Data:** new entities or fields, migrations, existing data
- **Architecture fit:** which layer/module, does it follow the documented patterns, any deviation and why
- **Security & permissions:** who can do this, what input is untrusted
- **Non-functional:** performance, concurrency, idempotency, retries
- **Dependencies:** other teams, services, tickets

**Think mode:** use the list above to find gaps, not as a script. Ask only about gaps, contradictions and risks in the user's text, checked against the docs and code. Ask questions; do not suggest answers. If the user is stuck, give a hint (a scenario to consider, a file to look at), not the answer.

When an answer is vague, push back with a concrete scenario ("what happens if two users do X at once?"). When you disagree with a decision, say so once with the reason, then accept the user's call.

When the user rules out an option, note it with their reason: it goes under Rejected options. Record only options that were actually discussed, never ones you make up afterwards.

Stop when every point above is either answered or consciously deferred.

## 4. Brief

Write the brief to `.ka/briefs/<slug>.md` and show it to the user for confirmation. In think mode, the brief records the user's decisions, not yours.

Set **ADR needed** to `yes` when a decision is hard to reverse, shapes architecture beyond this feature, or contradicts an existing ADR; otherwise `no`.

```markdown
# <Feature name>

**Mode:** think | ship
**ADR needed:** yes | no

## Goal

## In scope

## Out of scope

## Decisions

- <decision> — <why>

## Rejected options

- <option> — <why rejected>   (only options actually discussed; "none" if none)

## Edge cases & behaviour

## Data changes

## Architecture notes

## Open questions (deferred)

## Docs to update

## Docs read
```

Next step: `ka-split`. Optionally first `ka-docs rfc` to share the brief, or `ka-docs adr` when **ADR needed** is `yes`.

## Metrics

`.ka/metrics.jsonl` stays local and is never committed. Before appending, run `git check-ignore -q .ka/metrics.jsonl`. If it exits non-zero, tell the user to run `ka-init` in the project root, then append anyway.

Append one line to `.ka/metrics.jsonl`:
`{"step":"grill","slug":"<slug>","date":"<ISO date>","mode":"think|ship"}`
