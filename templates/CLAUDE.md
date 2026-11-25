# <Project name>

<One paragraph: what the project is and who uses it.>

## Docs map

Read the docs relevant to a task before planning it. Mark which ones you read.

| Path | What it covers |
|---|---|
| `docs/ARCHITECTURE.md` | Layers, modules, boundaries, patterns |
| `docs/CONVENTIONS.md` | Naming, code style, testing, commits |
| `.github/pull_request_template.md` | PR description template |
| `docs/adr/` | Architecture decisions (ka-docs adr) |
| `docs/GLOSSARY.md` | Domain terms (ka-docs glossary) |
| `docs/runbooks/` | Operational runbooks (ka-docs runbook) |
| `<add more>` | `<what it covers>` |

## Commands

Raw commands. The workflow skills wrap them with `ka-run` themselves.

```bash
test:      <e.g. pnpm test>
typecheck: <e.g. pnpm tsc --noEmit>
lint:      <e.g. pnpm lint>
```

## Workflow

This project uses Kansoo.
Feature: ka-grill → [ka-docs rfc|adr] → ka-split → ka-plan → ka-build → ka-review → ka-commit → ka-pr → ka-notes
Bug: ka-debug → ka-review → ka-commit → ka-pr → ka-notes → [ka-docs postmortem]
