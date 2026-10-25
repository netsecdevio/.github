# netsecdevio CI standards

Every repository calls shared workflows from `netsecdevio/.github` instead of
carrying its own copy. Fix a check once here and all 35 repos inherit it.

## Adopting CI in a repo

Add `.github/workflows/ci.yml` and call the workflow for the language:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

permissions:
  contents: read
  pull-requests: read

jobs:
  hygiene:
    uses: netsecdevio/.github/.github/workflows/reusable-hygiene.yml@main
  quality:
    uses: netsecdevio/.github/.github/workflows/reusable-python.yml@main
```

The same files are offered as starter workflows under **Actions > New workflow**
in any org repo.

## What each workflow does

| Workflow | Checks |
| --- | --- |
| `reusable-hygiene.yml` | Conventional Commits PR title, `actionlint`, gitleaks secret scan |
| `reusable-python.yml` | `ruff check`, `ruff format --check`, `ty check`, `pytest` |
| `reusable-node.yml` | `npm ci`, then `lint`, `typecheck`, `test`, `build` if defined |
| `reusable-go.yml` | `gofmt`, `go vet`, `golangci-lint`, `go test -race` |
| `reusable-shell.yml` | `shellcheck` at `warning` severity and above |

Node steps use `npm run --if-present`, so a repo opts into a check by adding the
script to `package.json`. No CI edit required.

Python is standardised on **ruff** for lint and formatting and **ty** for type
checking, against **Python 3.13 or newer**. Both gate the merge. A repo that is
not clean yet can set `types-advisory: true` to report ty findings without
failing, but that is a temporary state to be removed, not a default.

CI covers only the languages a repo actually contains. There is no Rust
workflow because the checks are opt-in per repo: a repo gets a job only by
calling that workflow.

## Why the PR title is linted

Squash is the only permitted merge method org-wide and the squash subject is the
PR title, so the title becomes the permanent history. It must read as a
Conventional Commit: `fix(httpproxy): resolve launcher path`.

## Branch policy

The org ruleset **netsecdevio Source Code Standard** applies to every repo's
default branch: pull request required, squash only, signed commits, linear
history, conversation resolution, no force-push, no deletion.

Add required status checks per repo once its CI is green, so a red build blocks
the merge.
