# Reusable GitHub Actions Workflow

A common GitHub Actions workflow that can be reused by multiple repositories.

## Overview

This repository contains a reusable GitHub Actions workflow:

```text
.github/
└── workflows/
    └── common-ci.yml
```

The workflow is triggered with:

```yaml
on:
  workflow_call:
```

That means this workflow is designed to be **called by another GitHub Actions workflow** rather than being triggered directly by a `push` or `pull_request` event.

## What This Workflow Does

The reusable workflow performs common Python CI tasks:

1. Checks out the caller repository's code.
2. Sets up the requested Python version.
3. Installs dependencies from `requirements.txt`.
4. Runs the Python test suite with `pytest`.

## Reusable Workflow

The current workflow is:

```yaml
name: Common Python CI

on:
  workflow_call:
    inputs:
      python-version:
        description: "Python version to use"
        required: false
        type: string
        default: "3.12"

jobs:
  python-ci:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python-version }}

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Check Project files
        run: |
          pwd
          echo "-------------------"
          ls -la
          echo "--------------------"
          find . -maxdepth 2 -type f

      - name: Run tests
        run: PYTHONPATH=. pytest
```

> `Check Project files` was added as a learning/debugging step to show what files are available on the GitHub runner. It can be removed in a production workflow if it is no longer needed.

## Input

The workflow accepts one input:

| Input | Required | Default | Description |
|---|---|---|---|
| `python-version` | No | `3.12` | Python version used by `actions/setup-python` |

The value is received with:

```yaml
${{ inputs.python-version }}
```

For example, a caller can pass:

```yaml
with:
  python-version: "3.12"
```

## How It Is Used

A caller repository references this workflow with:

```yaml
jobs:
  call-common-ci:
    uses: Satyajeet-Sah/reusable-workflow/.github/workflows/common-ci.yml@main
    with:
      python-version: "3.12"
```

The reusable workflow then uses that value here:

```yaml
with:
  python-version: ${{ inputs.python-version }}
```

## Repository Relationship

```text
┌──────────────────────────────────────┐
│ reusable-caller-workflow-python      │
│                                      │
│ .github/workflows/ci.yml             │
└──────────────────┬───────────────────┘
                   │
                   │ uses
                   ▼
┌──────────────────────────────────────┐
│ reusable-workflow                    │
│                                      │
│ .github/workflows/common-ci.yml      │
│                                      │
│ python-ci                             │
│  ├── Checkout                        │
│  ├── Setup Python                    │
│  ├── Install dependencies            │
│  └── Run tests                       │
└──────────────────────────────────────┘
```

## Key Concepts Learned

- `workflow_call` makes a workflow reusable.
- `uses:` calls a reusable workflow.
- `with:` passes input values to the reusable workflow.
- `inputs.<name>` reads those values inside the reusable workflow.
- The reusable workflow can be stored in a separate repository.
- The reusable workflow checks out and works with the caller repository's code.
- A single common workflow can be shared by many application repositories.

## Example Use Case

The same reusable CI workflow can be used by multiple Python repositories:

```text
python-project-1 ──┐
python-project-2 ──┤
python-project-3 ──┼──> reusable-workflow
python-project-4 ──┤
python-project-5 ──┘
```

This avoids duplicating the same CI configuration in every repository.

## Repository

`Satyajeet-Sah/reusable-workflow`
