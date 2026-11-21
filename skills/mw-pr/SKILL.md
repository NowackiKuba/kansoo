---
name: mw-pr
description: Write a pull request description from the project's PR template, based on the ticket, plan, review and diff, and open the PR. Use whenever the user says "open a PR", "write the PR description", "pull request", or when commits are ready to open a PR. Runs before mw-wid in the Minimalistic Workflow.
model: haiku
disable-model-invocation: true
---

# mw-pr

A PR description lets the reviewer understand the change before reading the code.

## Steps

1. Find the PR template via the `CLAUDE.md` docs map, or at `.github/pull_request_template.md`. Use it exactly; fill every section. If there is no template, use the default below.
2. Read the ticket, the plan, the review result and `git log <base>..HEAD` plus the diff.
3. Fill the template:
   - What and why, in plain language, short
   - How to test, as concrete steps
   - Risks, migrations, config or env changes, breaking changes
   - Anything the reviewer should look at closely
4. Do not describe what the diff already shows line by line. Do not claim tests or checks that were not run.
5. Open the PR without waiting for confirmation: push the branch if it is not pushed yet, write the description to a temp file and run `gh pr create --base <default branch> --title "<conventional title>" --body-file <file>`. If the work has a ticket, end the title with its number in parentheses: a tracker ticket (e.g. Linear or Jira) as `feat(auth): add login (ABC-123)`, or a local `tickets/NNN-slug.md` from `mw-split` as `feat(auth): add login (007)`. No ticket, no number. If a PR for the branch already exists, update it with `gh pr edit --title "<title>" --body-file <file>` instead. Report the PR URL.

Next step: `mw-wid`.

## Default template

```markdown
## What

## Why

Refs: <ticket>

## How to test

1.

## Risks & notes

## Checklist

- [ ] Tests pass
- [ ] Docs updated
```
