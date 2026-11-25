---
name: ka-review
description: Review a diff (current changes by default, or a given branch, commit or range) against its ticket, plan, architecture and conventions, and checks which docs it made stale, scaling the review to the size and risk of the change by spawning reviewer subagents. Use whenever the user asks for a code review, "review this", "check the diff", "review branch X", "review commit abc123", or after ka-build or ka-debug finishes. Best run in a fresh session. Part of Kansoo.
model: opus
disable-model-invocation: true
---

# ka-review

Review like a senior colleague who did not write the code. Output only what to change and why.

## Core principle (think mode)

The user produces the first version of every decision (scope, plan, hypothesis, review findings). You critique it, ask questions and execute. Never present your own full solution first. Reveal your own version only after the user's version exists, or when the user explicitly asks for it.

## 1. Resolve the target

Use the target from `$ARGUMENTS` or the user's message:

| Input        | Diff                                                      |
| ------------ | --------------------------------------------------------- |
| nothing      | uncommitted changes: `git diff HEAD` (staged + unstaged)  |
| branch name  | `git diff <base>...<branch>` (base = main/master, or ask) |
| commit hash  | `git show <hash>`                                         |
| range `a..b` | `git diff a..b`                                           |

If the diff is empty, say so in one line and stop.

## 2. Gather context

- The diff and `git diff --stat` for the same target.
- The ticket and plan in `tickets/` and `.ka/plans/` matching the change (or the `ka-debug` report), and the brief named in the plan's `**Brief:**` line, if they exist. Brief decisions and edge cases count as part of the plan.
- `CLAUDE.md`, architecture and conventions docs, and every doc in the `CLAUDE.md` docs map that covers what the diff touches (including ADRs in `docs/adr/`, runbooks and `docs/GLOSSARY.md`).
- Stale docs already flagged: the brief's `## Docs to update`, the build report, or the debug report's `## Stale docs`.
- Surrounding code of every changed file, not only the changed lines.

Use the mode recorded in the plan, ticket or debug report. If none is recorded, ask once: "think or ship?"

**Manual check.** If the ticket has a `## How to verify` section, ask the user whether they completed those steps before running any reviewer. If not, list the steps and wait until they confirm.

## 3. The user reviews first (think mode)

Before running any reviewer, ask the user for their own findings on the diff, one per line: `file:line — what — why`. "None" is a valid answer. Wait for it. Do not hint at problems before you have it.

Do not pass the user's findings to the reviewer subagents; they review independently.

## 4. Pick the review size

Count changed lines (added + removed, excluding lockfiles, generated files and snapshots) and files.

**Sensitive** if the diff touches any of the "Sensitive areas" list in `~/kansoo/README.md`.

| Size | When                                  | Reviewers                                                                                                             |
| ---- | ------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| S    | ≤ 100 lines, ≤ 3 files, not sensitive | review yourself, no subagents                                                                                         |
| M    | ≤ 400 lines, not sensitive            | 2 subagents: **correctness** (logic, edge cases, tests), **design** (architecture, conventions, plan fidelity)        |
| L    | > 400 lines, or any sensitive change  | 4 subagents: **correctness**, **security**, **design**, **data & tests** (migrations, queries, indexes, test quality) |

State the chosen size in one line before starting.

## 5. Run the reviewers

Give every subagent: the diff, the ticket/plan, the relevant docs, its single focus area, and this instruction:

> Report only real problems in your focus area. For each: `file:line`, severity (blocker, major, minor or nit, as defined below), what to change, why (one sentence), and evidence. No praise, no summaries, nothing a linter would catch. If you find nothing, return "none".
>
> Every finding must be proven from this codebase. No evidence, no finding:
> - **Bug or edge case:** a concrete failure scenario that traces through real code: "input X reaches `a.ts:42`, `b.ts:17` does Z, result is Y". A failing test or command output is better. "Might", "could" or "consider" is not evidence.
> - **Convention or design:** at least one existing place in the codebase that does it the other way, with `file:line`. If no code here shows the convention, it is a preference: drop it.
> - **Plan or ticket violation:** quote the plan line or acceptance criterion it breaks.
>
> Severity:
> - **blocker:** must be fixed before merge: a bug, security issue, data risk, broken acceptance criterion or plan violation.
> - **major:** should be fixed in this change: it will cause real pain in the next change to this code (a design or convention break, a missing or hollow test for changed behaviour, an unhandled edge case outside the acceptance criteria).
> - **minor:** worth fixing, fine as a follow-up: local readability, naming, small duplication.
> - **nit:** optional polish, the author's call.
>
> Pure preference with no evidence is not a nit: drop it. There is no minimum number of findings.

Run subagents in parallel. For size S, apply the same rules yourself across all areas.

## 6. Merge and verify

- Deduplicate overlapping findings.
- Verify every finding's evidence yourself: open each cited `file:line` and confirm the code there says what the finding claims, and walk the failure scenario through it. Drop the finding if a citation is wrong, the scenario does not hold, or it has no evidence.
- Run tests, typecheck and lint through `ka-run` (`ka-run test -- <test command>`, and so on). Do not trust earlier reports. Rerun a single failing test with full output only when the compact output is not enough to diagnose it.
- Check each finding's severity against the definitions in step 5 and correct it if the evidence does not support it.
- **Think mode:** check every user finding against the code the same way. Calling a user finding a false alarm needs evidence too: cite the code that shows it is not a problem. Match the user's findings to the verified ones: a verified finding the user reported is **found**, one they did not report is **missed**, and a user finding that is not a real problem is a **false alarm**.

## 7. Docs check

Do this yourself, after the code findings; reviewer subagents do not check docs.

For each doc gathered in step 2, decide: still true, needs an update, or a new doc is needed. Tick off every item in the brief's `## Docs to update`: each one is either already done in the diff or becomes a docs finding. For an ADR the brief or plan names, check that the code still matches the decision it records.

Every docs finding cites both sides, or it is dropped:

- the stale doc line (`docs/x.md:12`, quoted), and
- the diff line that made it stale (`src/y.ts:40`).

Severity: **blocker** = the doc now gives a wrong command, env var, config key or API (path, parameter, response), so someone following it will fail. **major** = it describes behaviour or a decision that no longer holds. **minor** = outdated wording, a missing mention, or a new doc worth adding. **nit** = phrasing only.

## 8. Output

Nothing beyond this format. No intro, no recap of the change, no closing remarks.

Ship mode:

```
Size: S|M|L · Checks: tests ✓/✗ · typecheck ✓/✗ · lint ✓/✗

Blockers
- path/file.ts:42 — <what to change> — <why>
  evidence: <cited file:line and what it shows>

Major
- path/file.ts:60 — <what to change> — <why>
  evidence: <cited file:line and what it shows>

Minor
- path/file.ts:88 — <what to change> — <why>
  evidence: <cited file:line and what it shows>

Nits
- path/file.ts:95 — <what to change>

Docs findings
- [blocker] docs/setup.md:12 — <what to change> — stale since src/config.ts:40
  doc says: "<quoted doc line>"
- [minor] docs/adr/0003-queue.md:20 — <what to change> — stale since src/queue.ts:15
  doc says: "<quoted doc line>"
```

Think mode (sort by severity within each section, blockers first):

```
Size: S|M|L · Checks: tests ✓/✗ · typecheck ✓/✗ · lint ✓/✗

You found
- [blocker] path/file.ts:42 — <what to change> — <why>
  evidence: <cited file:line and what it shows>

You missed
- [major] path/file.ts:88 — <what to change> — <why>
  evidence: <cited file:line and what it shows>

False alarms
- path/file.ts:10 — <the user's finding> — <why it is not a problem>
  evidence: <cited file:line and what it shows>

Docs findings
- [blocker] docs/setup.md:12 — <what to change> — stale since src/config.ts:40
  doc says: "<quoted doc line>"
```

Every finding except a nit keeps its evidence line, so the user can check it in seconds. Omit an empty section. If there are no findings at all, output the first line and `No issues found.`

**Applying docs fixes.** If there are docs findings, end with one question: `Apply the docs fixes?` On yes, make minimal edits to exactly those lines (or write the new doc), add any new doc to the `CLAUDE.md` docs map, and touch nothing else. Code findings are never applied here.

## Metrics

`{"step":"review","ticket":"NNN","date":"<ISO date>","mode":"think|ship","blockers":<count>,"major":<count>,"minor":<count>,"nit":<count>,"user_found":<count>,"user_missed":<count>}`

In ship mode, omit `user_found` and `user_missed`.
