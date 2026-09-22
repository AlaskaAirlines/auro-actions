# auro-actions

Reusable GitHub workflows and composite actions. Every Auro repo's build, test, and release runs from here.

## How it's used

A repo calls a workflow from here with GitHub's reusable workflow syntax:

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
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

Callers pin `@main`, not a tag. A breaking change merged here reaches every repo's CI right away, so test before you release.

Every workflow has a copy-paste caller example in [examples.md](./examples.md).

## Testing changes on dev

Push a change to the `dev` branch and test it before it reaches `@main`. The `auro-devops-*` repos pin `@dev`, so they run your in-progress workflows first:

- [auro-devops-formkit](https://github.com/AlaskaAirlines/auro-devops-formkit): monorepo test bed
- [auro-devops-component](https://github.com/AlaskaAirlines/auro-devops-component): single-component test bed

[auro-devops-library](https://github.com/AlaskaAirlines/auro-devops-library) is a third test bed. It drove the npm trusted publisher upgrade and will move to these workflows on `@dev` too.

Merge to `main` once `dev` passes there.

## Workflows

| Workflow | Runs on | Does |
|----------|---------|------|
| `pull-request.yml` | PR into dev | build, test, publish a dev package, deploy a preview |
| `release-candidate.yml` | push to dev | runs `auro rc-workflow` to cut the RC branch and PR |
| `release.yml` | push to `rc/**` or `main` | runs semantic-release to publish |
| `check-commits.yml` | PR | commit-lints and labels the PR with `semantic-status:` |
| `check-pr.yml` | PR review | requires 2 approvals on `semantic-status: breaking` PRs |
| `sync-tags.yml` | published release | fast-forward merges the release tag back into dev |
| `dev-demo.yml` | push to dev | deploys the dev demo |
| `no-build_pull-request.yml`, `no-build_release.yml` | same as above | drop-in versions for repos with no build step |

## Composite actions

In `.github/actions/`: `build`, `test`, `check-commits`, `pr-preview`. The reusable workflows call these for the shared steps.

## Required secrets

- **NPM_TOKEN**: npm auth for publishing. Needed by `pull-request.yml` and `release.yml`.
- **GH_TOKEN** (a PAT): optional, for the RC workflow. Without it, an RC branch can get created but never release.
- **GITHUB_TOKEN**: auto-provided. Used for previews, labels, releases, and the tag sync.

## A note before you change anything

Changes here affect every Auro repo. Test thoroughly before you release.
