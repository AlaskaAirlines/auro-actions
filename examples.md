# Workflow Usage Examples

How to call each reusable workflow from your repo. Callers pin `@main`, not a tag. The `auro-devops-*` test repos pin `@dev` to exercise in-progress changes before they reach `@main`.

Some workflows need secrets set in your repo:

- **NPM_TOKEN**: publishing to npm. Used by `pull-request`, `release`, and the no-build variants.
- **GH_TOKEN**: a PAT for the RC workflow. Optional. Without it the RC branch gets created but never releases.
- **CHROMATIC_PROJECT_TOKEN**: Chromatic visual tests.

---

## `pull-request.yml`

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

## `release-candidate.yml`

```yaml
name: RC Workflow

on:
  workflow_dispatch:
  push:
    branches:
      - 'dev'

jobs:
  action:
    uses: AlaskaAirlines/auro-actions/.github/workflows/release-candidate.yml@main
    secrets:
      GH_TOKEN: ${{ secrets.GH_TOKEN }}
```

---

## `release.yml`

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

---

## `check-commits.yml`

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

## `check-pr.yml`

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

## `sync-tags.yml`

```yaml
name: Sync Tags

on:
  release:
    types: [published]

jobs:
  action:
    uses: AlaskaAirlines/auro-actions/.github/workflows/sync-tags.yml@main
```

---

## `dev-demo.yml`

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

## `chromatic.yml`

```yaml
name: Chromatic

on:
  workflow_dispatch:
  pull_request:
    types: [opened, synchronize]
    branches-ignore:
      - 'rc/**'
      - 'main'
  push:
    branches:
      - dev

jobs:
  action:
    uses: AlaskaAirlines/auro-actions/.github/workflows/chromatic.yml@main
    secrets:
      CHROMATIC_PROJECT_TOKEN: ${{ secrets.CHROMATIC_PROJECT_TOKEN }}
```

---

## `codeql.yml`

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

## `add-project.yml`

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

## No-build variants

Drop-in replacements for repos with no build step. Same triggers as `pull-request.yml` and `release.yml`, just point at these instead.

### `no-build_pull-request.yml`

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
    uses: AlaskaAirlines/auro-actions/.github/workflows/no-build_pull-request.yml@main
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### `no-build_release.yml`

```yaml
name: Release Workflow

on:
  push:
    branches:
      - 'rc/**'
      - 'main'

jobs:
  action:
    uses: AlaskaAirlines/auro-actions/.github/workflows/no-build_release.yml@main
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```
