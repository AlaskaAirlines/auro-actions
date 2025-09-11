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
  pr-workflow:
    uses: AlaskaAirlines/auro-actions/.github/workflows/pull-request.yml@main
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
      AURO_SURGE_TOKEN: ${{ secrets.AURO_SURGE_TOKEN }}
```

## Required Secrets

Some workflows require specific secrets to be configured in your repository:

- **NPM_TOKEN**: Required for `pull-request.yml` and `release.yml` workflows for npm operations
- **AURO_SURGE_TOKEN**: Required for `pull-request.yml` workflow for Surge deployment

## Available Workflows

- Build and test workflows
- Automated release processes
- Deploy and publish workflows

## Contributing

Changes to workflows in this repository affect all Auro repositories. Test thoroughly before releasing updates.
