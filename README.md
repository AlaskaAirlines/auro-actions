# auro-actions

Centralized repository for reusable GitHub Actions workflows and composite actions used across all Auro Design System repositories.

## What's Included

- **Reusable Workflows**: Standardized CI/CD workflows for build, test, and release processes
- **Composite Actions**: Custom GitHub Actions for common Auro development tasks

## Usage

Auro repositories consume workflows from this repository using GitHub's reusable workflow syntax:

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
      component: kabab-case-name
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
      AURO_SURGE_TOKEN: ${{ secrets.AURO_SURGE_TOKEN }}
```

## Required Secrets

Some workflows require specific secrets to be configured in your repository:

- **NPM_TOKEN**: Required for `pull-request.yml` and `release.yml` workflows for npm operations
- **AURO_SURGE_TOKEN**: Required for `pull-request.yml` workflow for Surge deployment
- **GH_TOKEN**: Required for `sync-tags.yml` and `release-candidate.yml`. Must belong to a member of the `dev` branch ruleset's bypass team — the ruleset requires a pull request, so the default `GITHUB_TOKEN` cannot push to `dev`. If this token expires the sync stops working.
- **ADO_TOKEN**: Required for `release-candidate.yml`. Azure DevOps PAT (org `itsals`) with Work Items: Read, used to resolve the `auro-rcs` Release ticket that names the RC branch.

## `custom-elements.json` package field

`pull-request.yml` sets `"package": "<name>@<version>"` in `custom-elements.json` before publishing a PR release. For `release.yml`, the version is only known to semantic-release, so each consuming repo must set the field itself with `@semantic-release/exec`. List it **after** `@semantic-release/npm` and before any publish step:

```json
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/npm",
    ["@semantic-release/exec", { "prepareCmd": "node scripts/add-package-to-manifest.mjs" }],
    "@semantic-release/github"
  ]
}
```

```js
// scripts/add-package-to-manifest.mjs
import fs from 'node:fs';

const file = './custom-elements.json';
if (fs.existsSync(file)) {
  const { name, version } = JSON.parse(fs.readFileSync('./package.json', 'utf8'));
  const manifest = JSON.parse(fs.readFileSync(file, 'utf8'));
  manifest.package = `${name}@${version}`;
  fs.writeFileSync(file, `${JSON.stringify(manifest, null, 2)}\n`);
}
```

`@semantic-release/npm` writes the release version into `package.json` in its own prepare step, which is why the exec plugin must come after it.

## Available Workflows

- Build and test workflows
- Automated release processes
- Deploy and publish workflows

## Git flow and releases

auro-actions follows the same git flow as Auro components. Every Auro repository calls these workflows at `@main`, so a change reaches consumers only when its release candidate is merged to `main`.

1. **Feature branch → `dev`.** Branch from `dev` and open a pull request into `dev`. Commits must follow [Conventional Commits](https://www.conventionalcommits.org/) and reference the ADO work item (`AB#<id>`). The `validate` check (`npm run validate`), commit check, review policy and CodeQL run on the pull request.
2. **`dev` → release candidate.** Each push to `dev` runs `auro rc-workflow`. It finds the `auro-rcs` Release ticket linked to the new commits, then creates or updates the `rc/<ticket id>` branch and the `RC #<ticket id>` pull request into `main`. If no commit links to a Release ticket, no RC is created.
3. **`rc/*` → prerelease.** Each push to an `rc/*` branch runs semantic-release, which creates a GitHub prerelease such as `v1.2.0-rc-<ticket id>.1`. To try a release candidate from a consuming repository, point its workflow at `@rc/<ticket id>`.
4. **RC pull request → `main`.** Merge with a merge commit (never squash or rebase). semantic-release creates the `vX.Y.Z` tag and GitHub release. Nothing is published to npm; the package is private.
5. **`main` → `dev`.** The sync-tags workflow fast-forwards `dev` to `main`. If `dev` has diverged, merge `main` into `dev` with a merge commit (never a rebase) and re-run it.

### Choosing a commit type

The release version comes from commit types, and a release candidate is created whether or not its commits trigger a release. Because almost every change here touches CI files, pick the type by **who the change affects**, not by which files it touches:

| Change | Type | Release |
|---|---|---|
| A reusable workflow or composite action change that consuming repositories will see (new input, behavior change, bug fix) | `feat` / `fix` | minor / patch |
| A change consumers must act on (removed or renamed input, secret or workflow) | any type with `BREAKING CHANGE:` | major |
| A change to auro-actions' own pipeline (`repo-*.yml`, `.releaserc`, hooks, docs) | `ci` / `docs` / `chore` | none |

An RC made up only of `ci`, `docs` or `chore` commits still merges to `main` (and so reaches consumers on `@main`), but it creates no tag, release or changelog entry. If a change reaches consumers, use `feat` or `fix` so it gets a version that can be tracked and rolled back. When needed, a maintainer can change a commit's type to force a release.

This repository runs its own workflows from the `repo-*.yml` files in `.github/workflows`. They call the reusable workflows through local paths (`./.github/workflows/…`), so a pull request tests its own workflow changes. Composite actions are still referenced at `@main` inside the reusable workflows, so changes to `.github/actions` only take effect after they reach `main`.

## Contributing

Changes to workflows in this repository affect all Auro repositories. Test thoroughly before releasing updates. See [CONTRIBUTING](.github/CONTRIBUTING.md) for branch, commit and pull request guidelines.
