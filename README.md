# github-actions

Reusable GitHub Actions workflows for my projects.

This repository is the central place for automation shared across repositories. Projects keep only a small caller workflow; the actual update logic lives here.

## Workflows

### `update-npm-dependencies.yml`

Safely refreshes npm dependencies and opens a pull request.

Default behaviour:

- patch updates only
- Node.js 20.x
- keeps `package-lock.json` instead of deleting it
- verifies the new lockfile with a clean install
- runs `check-types` and `test` when those npm scripts exist
- build verification is opt-in because builds often require project-specific environment variables
- opens a pull request only after validation succeeds
- never auto-merges dependency changes

Minor updates can be enabled explicitly by the caller. Major updates are intentionally not supported by the automated workflow and should be reviewed manually.

## Usage

The schedule belongs to the consuming repository. A minimal caller looks like this:

```yaml
name: Update dependencies

on:
  workflow_dispatch:
  schedule:
    - cron: "0 0 1 * *"

permissions:
  contents: write
  pull-requests: write

jobs:
  update:
    uses: mixu-94/github-actions/.github/workflows/update-npm-dependencies.yml@main
```

Optional inputs can be supplied with `with:`:

```yaml
jobs:
  update:
    uses: mixu-94/github-actions/.github/workflows/update-npm-dependencies.yml@main
    with:
      node-version: "20.x"
      update-target: patch
      run-tests: true
      run-typecheck: true
      run-build: false
```

Once the workflow has settled, callers should preferably pin it to a version tag or commit SHA instead of `@main`.

## Third-party actions

This repository currently uses maintained upstream actions rather than reimplementing GitHub plumbing from scratch. Their versions are centralized here and kept up to date with Dependabot.

If we later fork or copy code from another action, its license and attribution must be checked first.

## Design rules

1. Automations should be conservative by default.
2. Never auto-merge dependency upgrades.
3. Major dependency upgrades stay manual.
4. Validate before creating a pull request.
5. Give workflows only the permissions they need.
6. Centralize shared logic instead of copying it into every project.
