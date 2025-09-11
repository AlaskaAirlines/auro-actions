# auro-actions

Centralized repository for reusable GitHub Actions workflows and composite actions used across all Auro Design System repositories.

## What's Included

- **Reusable Workflows**: Standardized CI/CD workflows for build, test, and release processes
- **Composite Actions**: Custom GitHub Actions for common Auro development tasks

## Usage

Auro repositories consume workflows from this repository using GitHub's reusable workflow syntax:

```yaml
uses: AlaskaAirlines/auro-actions/.github/workflows/[workflow-name].yml@v1
with:
  # Repository-specific inputs
  node-version: '22'
  npm-token: ${{ secrets.NPM_TOKEN }}
```

## Available Workflows

- Build and test workflows
- Automated release processes
- Deploy and publish workflows

## Contributing

Changes to workflows in this repository affect all Auro repositories. Test thoroughly before releasing updates.
