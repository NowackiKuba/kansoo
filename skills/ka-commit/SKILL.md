---
name: ka-commit
description: Create small, logical conventional commits for the current changes, with messages that explain why, then push the branch. Use whenever the user says "commit", "commit this", "make commits", or when changes are ready to commit after review. Part of Kansoo.
model: haiku
disable-model-invocation: true
allowed-tools: Bash(git *), Read, Edit
---

# ka-commit

Commits tell the story of the change to the next person reading `git log` or `git blame`.

## Steps

1. Read the commit rules in the conventions doc. They override the defaults below.
2. Run `git status` and `git diff` (staged and unstaged). Read the ticket or plan for the "why".
3. Group changes into logical commits: each one builds and makes sense alone (e.g. migration, then domain logic with its tests, then API). Do not mix refactors with behaviour changes. Do not split a change from its own tests. Never commit `.ka/metrics.jsonl`.
4. Stage per commit (`git add <files>`, `git add -p` for split files) and commit. Do not wait for confirmation.
5. Push: `git push`, or `git push -u origin HEAD` if the branch has no upstream. Report the commits and the pushed branch.

If you are on the default branch, stop before committing and ask. Never commit secrets, `.env` files, build output or unrelated changes; leave them unstaged and point them out. Never force-push, amend or rewrite commits that may already be pushed unless the user asks.

## Default format (Conventional Commits)

```
<type>(<scope>): <imperative summary, max ~72 chars>

<why this change, what problem it solves, anything non-obvious>

Refs: <ticket>
```

Types: feat, fix, refactor, test, docs, chore, perf, build, ci.

**Example**

```
fix(invoices): prevent duplicate KSeF submission on retry

The BullMQ job retried after a timeout even when KSeF had already
accepted the invoice. Check submission status before resending.

Refs: 014
```
