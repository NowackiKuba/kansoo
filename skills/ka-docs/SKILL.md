---
name: ka-docs
description: Write a project doc from what the workflow already decided — an RFC or ADR from a brief, a postmortem after a bug fix, a runbook, or glossary entries. Use whenever the user says "ka-docs", "write an RFC", "write an ADR", "record this decision", "write a postmortem", "write a runbook", "add to the glossary". Optional step right after ka-grill (rfc, adr) or after ka-debug (postmortem) in Kansoo.
model: opus
disable-model-invocation: true
---

# ka-docs

Turn decisions that already exist into a committed doc other people can read. This skill formats and checks; it never decides.

Usage: `ka-docs <rfc|adr|postmortem|runbook|glossary> [slug]`. If the type is missing, ask which one.

## Core principle

No modes. The user already made the decisions in `ka-grill` (or `ka-debug`); this skill drafts the doc from that record and never asks the user to re-decide or re-write it. If a decision the doc needs is missing from the brief, stop and send the user back to `ka-grill` instead of deciding it here or asking for it ad hoc.

## Rules for every type

- **Source only.** Every statement comes from the brief (`.ka/briefs/<slug>.md`), the plan, the debug report, the code or the user. Never invent decisions, alternatives, numbers, dates or owners. If something the format asks for is unknown, ask; if the user does not know, write `unknown` rather than a guess.
- **Self-contained.** These docs are committed and read by people outside this session. Never link to `.ka/` or `tickets/` (they are local and excluded from git); carry the content over instead.
- **Confirm before writing.** Show the draft, wait for approval, then write the file. Edit an existing doc in place; never rewrite parts the change does not touch.
- **Docs map.** After writing, make sure `CLAUDE.md`'s docs map has a row for the doc's folder (`docs/rfc/`, `docs/adr/`, `docs/postmortems/`, `docs/runbooks/`) or for `docs/GLOSSARY.md`, so `ka-grill` and `ka-review` read it. Add the row if it is missing.
- Read `CLAUDE.md` and the existing docs of the same type first, and match their format if the project already has one.

## rfc

The shareable version of a brief, for people who were not in the grill.

Input: the brief. If there is none, stop and suggest `ka-grill`.

The brief already holds the user's decisions; only restructure and rewrite for an outside reader. If a reader could not follow a decision because the brief does not record it, send the user back to `ka-grill`.

Write `docs/rfc/<slug>.md`:

```markdown
# RFC: <Feature name>

**Status:** Draft
**Author:** <git config user.name>
**Date:** <ISO date>

## Summary
<2–3 sentences>

## Motivation
<goal: who it is for, what problem, how we will know it works>

## Proposal
<in scope and the decisions, each with its why>

## Out of scope

## Alternatives considered
<the brief's Rejected options, each with why; "None were recorded." if empty>

## Edge cases & behaviour

## Open questions
```

Next step: `ka-docs adr` if the brief says **ADR needed: yes**, otherwise `ka-split`.

## adr

One ADR per decision that is hard to reverse or shapes architecture beyond this feature.

Input: the brief (and the RFC or plan, if they exist). If the brief says **ADR needed: no**, say so and ask whether the user still wants one.

1. Pick the decision. If the brief has several candidates, list them and let the user choose; one ADR each.
2. Draft the **Decision** and its **rationale** from the brief, keeping the user's wording from the grill. If the brief records the decision but not why, send the user back to `ka-grill`. Flag (do not fix) a draft that contradicts an existing ADR or a claim the code does not support (cite `file:line`).
3. **Alternatives come only from the brief's Rejected options** (or the RFC's Alternatives considered). Never add an option that was not discussed. If none were recorded, write `No alternatives were recorded.`
4. Number it: the next free `NNNN` in `docs/adr/`.
5. If it replaces an existing ADR, set that one's status to `Superseded by NNNN` (after the user confirms).

Write `docs/adr/NNNN-<decision-slug>.md`:

```markdown
# NNNN. <Decision as a short statement>

**Status:** Proposed
**Date:** <ISO date>

## Context
<the problem and the forces at play>

## Decision
<what we will do>

## Alternatives considered
- <option> — <why rejected>

## Consequences
<what gets easier, what gets harder, what we now have to maintain>
```

The status stays **Proposed**: `ka-pr` sets it to **Accepted** when the change that implements it ships, and `ka-review` checks the code still matches it.

## postmortem

After `ka-debug`, for a bug that reached users. Blameless: describe systems and decisions, never people.

Input: the debug report, the fix commits or PR, and the user for impact and timeline.

Draft Root cause and Fix from the debug report; ask the user for Impact, Timeline and Action items, which only they know. Flag an action item that would not have prevented this, or a missing detection gap.

Write `docs/postmortems/<ISO date>-<slug>.md`:

```markdown
# Postmortem: <short title>

**Date:** <ISO date> · **Severity:** <user's call>

## Summary

## Impact
<who was affected, how, for how long; numbers only from the user or real data>

## Timeline
- <time> — <event>

## Root cause
<from the debug report>

## Fix
<commit or PR, and the reproduction test that now guards it>

## Detection
<how we found out, and how we could have found out sooner>

## Action items
- [ ] <action> — <owner or "unassigned">
```

## runbook

Steps someone on call can follow without knowing the code.

Input: the user's description of the situation it covers, plus the code and config it touches.

Draft the steps from the code and config.

Every command, script, env var, config key and URL must exist in the repo or come from the user: cite where you found it. Anything you cannot confirm becomes `TODO [human]: <what is missing>`, never a plausible guess.

Write `docs/runbooks/<slug>.md`:

```markdown
# Runbook: <situation>

## When to use
<the alert, symptom or request that triggers it>

## Prerequisites
<access, tools, permissions>

## Steps
1. <exact command or action>

## Verify
<how to tell it worked>

## Rollback

## Escalation
<who to contact when the steps do not help>
```

## glossary

Domain terms, so the team, the docs and the code use the same words.

Input: the brief and the code it names, or terms the user gives.

1. Collect candidate terms: domain words from the brief, and names the code uses for the same concepts. Flag any concept with two names (in the brief vs the code, or two places in the code), with `file:line`.
2. Draft definitions from the brief and code.
3. The user confirms which terms go in. Do not add generic programming terms.

Add to `docs/GLOSSARY.md` (create it if missing), keeping entries alphabetical:

```markdown
# Glossary

**<Term>** — <one-sentence definition>. In code: `<IdentifierName>`.
```
