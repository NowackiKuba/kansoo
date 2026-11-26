<div align="center">

# Kansoo

**Each skill does one thing. You approve the plan; the agent ships.**

<img src="assets/hero.webp" alt="Animated demo: the tagline types out, the ka-* skill pipeline lights up, then /ka-plan waits for approval and /ka-build passes test, typecheck and lint" width="800">

![Claude Code skills](https://img.shields.io/badge/Claude_Code-skills-D97757?style=flat-square)
![11 skills](https://img.shields.io/badge/skills-11-555?style=flat-square)
![Bash scripts](https://img.shields.io/badge/scripts-bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white)

[kansoo.io](https://kansoo.io)

</div>

A small set of Claude Code skills for shipping features and fixing bugs end to end. The human approves the plan; the agent implements, verifies and documents.

## Quick start

```bash
git clone https://github.com/NowackiKuba/kansoo.git ~/kansoo
cp -r ~/kansoo/skills/* ~/.claude/skills/
ln -sfn ~/kansoo/bin/* ~/.local/bin/

cd your-project && ka-init   # keep tickets/ and .ka/ out of git
```

Then in Claude Code: `/ka-design` once per project, `/ka-grill` to start a feature, `/ka-debug` for a bug. Full setup: [Install](#install).

## Flows

**Once per project:** `ka-design` before the first feature

```mermaid
flowchart LR
  subgraph Feature
    direction LR
    grill[ka-grill] -. optional .-> docs[ka-docs rfc / adr] -.-> split
    grill --> split[ka-split] --> plan[ka-plan] --> approve{{you approve}} --> build[ka-build]
  end
  subgraph Bug
    direction LR
    debug[ka-debug]
  end
  build --> review[ka-review]
  debug --> review
  review -. code findings .-> build
  review --> commit[ka-commit] --> pr[ka-pr] --> notes[ka-notes]
```

**Feature:** `ka-grill` → (`ka-docs rfc` / `ka-docs adr`, optional) → `ka-split` → `ka-plan` → `ka-build` → `ka-review` → `ka-commit` → `ka-pr` → `ka-notes`

**Bug:** `ka-debug` → `ka-review` → `ka-commit` → `ka-pr` → `ka-notes`, then `ka-docs postmortem` if the bug reached users

`ka-review` also checks which docs the change made stale and offers to fix them, in both flows. Code findings go back to `ka-build` (it reads the review saved in `.ka/reviews/`), then `ka-review` runs again as a small re-review of just the fixes.

| Skill | Does |
|---|---|
| `ka-design` | Grills the visual design system, writes `docs/DESIGN.md` and design tokens |
| `ka-grill` | Reads docs and code, asks questions until the feature is unambiguous, writes a brief, flags whether an ADR is needed |
| `ka-split` | Turns a brief into local tickets in `tickets/` |
| `ka-plan` | Turns a ticket into a plan and stops for approval |
| `ka-build` | Implements the approved plan, or a review's code findings, with a test/typecheck/lint loop |
| `ka-debug` | Reproduces a bug with a failing test, then fixes it |
| `ka-review` | Reviews the diff against the ticket, plan and conventions, plus docs findings for docs the change made stale |
| `ka-docs` | Optional: RFC or ADR from the brief, postmortem after a bug, runbook, glossary |
| `ka-commit` | Small, logical conventional commits, then push |
| `ka-pr` | Opens the PR with a description from the project's PR template, accepts the ADRs it implements |
| `ka-notes` | Short "what I did" note in your Obsidian vault, one folder per project |

Every skill runs only when you invoke it (`/ka-*`); none triggers on its own.

`ka-grill`, `ka-plan`, `ka-debug`, `ka-review` and `ka-docs` run on opus; `ka-build` runs on sonnet. If build `attempts` or `stopped` results rise noticeably, switch ka-build back to opus.

## Core principle

The user produces the first version of every decision (scope, plan, hypothesis, review findings). The AI critiques it, asks questions and executes. The AI never presents its own full solution first. It may reveal its own version only after the user's version exists, or when the user explicitly asks for it.

## Think mode vs ship mode

`ka-grill`, `ka-plan` and `ka-debug` start by proposing a mode with a one-sentence reason, unless you already said or the ticket records one. Reply `ok` to accept, or `think` / `ship` to override. The mode is recorded in the brief, ticket, plan and debug report, so later steps follow it.

They propose **think** when the change touches a [sensitive area](#sensitive-areas), touches paths you have not changed in the last ~3 months (`git log --author`), or involves a design decision that is hard to reverse. Otherwise they propose **ship**.

```
Proposed mode: think — touches migrations and queue retries. OK?
```

| | **think** | **ship** |
|---|---|---|
| `ka-grill` | You write goal, scope, out of scope, edge cases, risks; the AI asks only about gaps | The AI drives the questions |
| `ka-split` | The AI proposes the split; you confirm | The AI proposes the split; you confirm |
| `ka-plan` | You write the plan; the AI reviews it, then offers its own version for comparison | The AI writes the plan |
| `ka-build` | Builds only what the plan specifies, stops on open design decisions, ends with a comprehension check | Builds the plan |
| `ka-debug` | You give the hypothesis and the fix; the AI hints in steps and critiques | The AI finds the cause and proposes the fix |
| `ka-review` | You list your findings first; output is You found / You missed / False alarms | Blockers / Major / Minor / Nits |

In both modes you approve before anything is written, and a plan section marked `[human]` is left for you to implement.

## Sensitive areas

The single definition. `ka-review` uses it to pick review size L; `ka-grill`, `ka-plan` and `ka-debug` use it to propose think mode.

- auth/permissions
- payments
- migrations or schema
- public API contracts
- security config
- infra/deploy
- queues/jobs with retries
- cryptography
- PII handling

## Manual verification

`ka-split` adds an optional `## How to verify` section to tickets with user-facing behaviour: manual steps to check the change in the UI or API. Before running reviewers, `ka-review` asks whether you did them, and lists them if not. This can later be automated with agent-browser.

## Files the workflow uses

<details>
<summary>Paths</summary>

```
CLAUDE.md                 # project map: docs, commands (see templates/CLAUDE.md)
tickets/NNN-slug.md       # local tickets (ka-split)
.ka/briefs/<slug>.md      # briefs (ka-grill)
.ka/plans/NNN-slug.md     # approved plans (ka-plan)
.ka/reviews/NNN.md        # code findings you kept in review; ka-build marks them applied, re-review deletes the file
.ka/metrics.jsonl         # one line per step, for measuring the workflow (local only)
docs/rfc/<slug>.md        # RFCs (ka-docs rfc), committed
docs/adr/NNNN-slug.md     # ADRs (ka-docs adr), accepted by ka-pr
docs/postmortems/         # postmortems (ka-docs postmortem)
docs/runbooks/            # runbooks (ka-docs runbook)
docs/GLOSSARY.md          # domain terms (ka-docs glossary)
```

</details>

## Install

```bash
git clone https://github.com/NowackiKuba/kansoo.git ~/kansoo
cp -r ~/kansoo/skills/* ~/.claude/skills/
```

Put the scripts in `bin/` on your PATH (`~/.local/bin` must be on PATH):

```bash
chmod +x ~/kansoo/bin/*
ln -sfn ~/kansoo/bin/* ~/.local/bin/
```

Then add the sections from `templates/CLAUDE.md` to your project's `CLAUDE.md`, and run `ka-init` in the project root.

<details>
<summary><b>Scripts</b>: ka-init, ka-run, ka-stats</summary>

| Script | Does |
|---|---|
| `ka-init [--commit-plans]` | Adds `tickets/` and `.ka/` to `.git/info/exclude` (local, never committed), so workflow files stay out of team repos. `--commit-plans` keeps `.ka/plans/` committable, for your own repos where you want plan history. |
| `ka-run <test\|typecheck\|lint> -- <command...>` | Runs a check with compact output: `ok (<kind>)` on success; on failure the last 40 lines (test) or the first 20 error lines plus an error count (typecheck, lint). Exits with the command's exit code. The skills run every check through it. |
| `ka-stats [weeks=4]` | Weekly summary of `.ka/metrics.jsonl` in the current project, answering the three questions below. Needs `jq`. |

</details>

## Metrics

<details>
<summary>What gets logged and why</summary>

`grill`, `plan`, `build`, `debug` and `review` each append one line to `.ka/metrics.jsonl`. The metrics answer exactly three questions; a field that answers none of them is not logged.

| Question | Fields |
|---|---|
| 1. Are my plans getting better? | plan `revisions`; build `deviations`, `attempts`, `result` (`stopped`) |
| 2. Am I catching more in review? | review `user_found` vs `user_missed` (think mode) |
| 3. Do I actually use think mode, and is it worth it? | `mode` on every line: think/ship ratio, and review `blockers` and build stops per mode |

</details>

## Principles

1. Fixed context lives in docs and skills; the task prompt carries only what is unique.
2. A human approves the plan before code is written. Fixing a plan is cheaper than fixing code.
3. Nothing is "done" while tests, typecheck or lint are red.
4. Review happens in a fresh session, so the reviewer does not share the author's assumptions.
5. The steps that matter log one metrics line, so you can tell whether the workflow actually helps (`ka-stats`). `.ka/metrics.jsonl` stays local: `ka-init` excludes it, and the steps that log metrics remind you to run it when the file is not ignored.
