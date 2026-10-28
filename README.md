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

Both `pull-request.yml` and `release.yml` set `"package": "<name>@<version>"` in `custom-elements.json` before publishing. No per-repo configuration is needed.

- `pull-request.yml` reads the name and version from `package.json` after `auro pr-release` has rewritten it.
- `release.yml` runs `npx semantic-release --dry-run` first and reads the version from its `The next release version is X` line, then stamps the file before the real release. If no release is due, the step is skipped.

The dry run needs the same `GITHUB_TOKEN` and `NPM_TOKEN` as the real release, and it depends on semantic-release's log wording. If a semantic-release upgrade changes that message, the field will silently stop being set.

## Version check (`api-diff.yml`)

`api-diff.yml` runs [`auro api-diff`](https://github.com/AlaskaAirlines/auro-cli#auro-api-diff-baseref-headref) on each pull request into `dev`. The check works out the smallest version bump the code changes require, then compares it with the bump the commits declare. If the commits declare less, the check fails. For example, a `fix:` commit that removes a public attribute fails, because removing an attribute requires a major release. Each run posts a PR comment listing the changes and commits.

The check is opt-in. To turn it on in a repo:

1. Make sure the repo's lockfile resolves `@aurodesignsystem/auro-config` 1.3.2 or later. Earlier versions install a commit parser that ignores the `!` breaking-change marker, so the check would fail correctly labeled `feat!:` commits.
2. Add the caller from [examples.md](./examples.md#9-api-diffyml).
3. Add `Version Check / api-diff` as a required status check in the repo's `Dev` ruleset.

**Override.** If the check is wrong, add the `semver-override` label to the pull request. The check still runs and comments, but it doesn't fail. `check-pr.yml` then requires an approving review, the same as for a breaking change. Explain in the PR description why the declared bump is correct.

**Pinned auro-cli version.** The workflow runs the auro-cli version in its `auro-cli-version` input (default `5.2.0`), not the latest release. A new auro-cli release can change the rules, so it reaches enforced repos only when this default is bumped here.

The check fails, without an override, when it can't run: for example, when the head doesn't build or the repo has no `@semantic-release/commit-analyzer` installed.

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
