
# Workflow Usage Examples

Below are examples of how to call each reusable workflow from another repository. Replace `main` with a specific tag or branch if needed.

**Note**: Some workflows require secrets to be passed. Make sure your repository has the necessary secrets configured:

- **NPM_TOKEN**: Required for workflows that publish to npm
- **AURO_SURGE_TOKEN**: Required for workflows that deploy to Surge

---

## 1. `add-project.yml`

```yaml
name: Add Project to Issue

on:

  issues:
    types: [opened]


jobs:
  action:
    uses: AlaskaAirlines/auro-actions/.github/workflows/add-project.yml@main
```

---

## 2. `check-commits.yml`

```yaml
name: Check Commits

on:
  pull_request:
    types: [opened, reopened, unlabeled, synchronize]

jobs:
  action:
    name: Check Commits
    uses: AlaskaAirlines/auro-actions/.github/workflows/check-commits.yml@main
```

---

## 3. `pull-request.yml`

```yaml
name: PR Workflow

on:
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches-ignore:
      - 'rc/**'
      - 'main'

jobs:
  action:
    uses: AlaskaAirlines/auro-actions/.github/workflows/pull-request.yml@main
    with:
      cache-dirs: |
        ./components/**/dist
        ./components/**/demo
        ./components/**/node_modules
        ./node_modules
        ./custom-elements.json
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## 4. `sync-tags.yml`

Fast-forwards `dev` to `main` after every release, so the two branches never drift apart.

Triggered by work landing on `main` rather than by the `release` event: releases are published by a bot account, and GitHub does not start workflows from bot-published releases. `workflow_dispatch` is included so the sync can also be run by hand.

```yaml
name: Sync Tags

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  action:
    uses: AlaskaAirlines/auro-actions/.github/workflows/sync-tags.yml@main
    secrets:
      GH_TOKEN: ${{ secrets.GH_TOKEN }}
```

---

## 5. `codeql.yml`

```yaml
name: Code QL

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  action:
    uses: AlaskaAirlines/auro-actions/.github/workflows/codeql.yml@main
```

---

## 6. `dev-demo.yml`

```yaml
name: Dev Demo

on:
  push:
    branches:
      - dev
  pull_request:

jobs:
  action:
    uses: AlaskaAirlines/auro-actions/.github/workflows/dev-demo.yml@main
```

---

## 7. `check-pr.yml`

```yaml
name: Review Policy

on:
  pull_request_review:
    types: [submitted, edited, dismissed]
  pull_request:
    types: [labeled, unlabeled, opened, synchronize, reopened]

jobs:
  action:
    uses: AlaskaAirlines/auro-actions/.github/workflows/check-pr.yml@main
```

---

## 8. `release.yml`

```yaml
name: Release Workflow

on:
  push:
    branches:
      - 'rc/**'
      - 'main'

jobs:
  action:
    uses: AlaskaAirlines/auro-actions/.github/workflows/release.yml@main
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
    with:
      cache-dirs: |
        ./dist
        ./demo
        ./node_modules
        ./custom-elements.json
```
