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

## Contributing

Changes to workflows in this repository affect all Auro repositories. Test thoroughly before releasing updates.
