# github-actions

Reusable GitHub Actions and workflows for my projects.

This repository is the central automation toolbox for my repositories. Individual projects keep small caller workflows; the reusable logic lives here.

## Why centralize this?

Without reusable workflows, every repository slowly grows its own copy of CI, release and maintenance YAML. Those copies drift apart and become annoying to update.

With this repository:

- one CI improvement can benefit many projects
- action versions are maintained in one place
- permissions and security rules stay consistent
- project repositories contain much less boilerplate
- each project still controls when a workflow runs and which secrets it receives

## Reusable workflows

### `ci-node.yml`

Shared CI for Node.js and Next.js-style projects. It supports clean installs, multi-Node builds, commit validation, linting, type checks, unit tests, Codecov, Playwright, Storybook, E2E tests and test artifacts.

Think of this workflow as the common project TÜV: repositories can be checked against the same quality rules without copying the implementation.

### `release-node.yml`

Shared semantic-release workflow. The caller decides *when* release is allowed; this repository defines *how* the release is performed consistently.

### `security.yml`

Shared security checks using GitHub Dependency Review and `zizmor`. Older/reference projects can start in observation mode and later opt into blocking security checks.

### `update-npm-dependencies.yml`

Conservative npm maintenance. Patch updates are the default, the lockfile is preserved and revalidated, tests/type checks can run before a PR is created, and major upgrades are kept manual.

## Composite actions

### `actions/create-pr`

Small in-house action that detects generated changes, creates a dedicated automation branch, commits them and creates or updates a pull request with GitHub CLI.

The automation branch is disposable state and must not be used for manual development work.

## Security model

This repository can indirectly receive powerful permissions from caller repositories, so it is treated as security-sensitive infrastructure.

Key rules:

- third-party actions are pinned to full immutable commit SHAs
- readable version numbers are kept as comments next to those SHAs
- Dependabot checks GitHub Action updates weekly
- permissions are kept as small as practical
- workflow changes are audited by the repository's own security checks
- write-capable automation never auto-merges changes

See `SECURITY.md` for vulnerability reporting and `CONTRIBUTING.md` for contribution rules.

## Example: Node CI caller

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  ci:
    uses: mixu-94/github-actions/.github/workflows/ci-node.yml@<commit-sha>
    with:
      build-node-versions: '["20.x","22.x"]'
      validate-commits: true
      install-playwright: true
      storybook-command: npm run test-storybook:ci
      e2e-command: npx percy exec -- npm run test:e2e
```

## Example: dependency updater caller

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
    uses: mixu-94/github-actions/.github/workflows/update-npm-dependencies.yml@<commit-sha>
```

## Versioning

Consumers should prefer a full commit SHA for maximum reproducibility. A moving major reference such as `v1` may also be provided as the human-friendly stable channel.

The intended model is:

- `main` = current development/stable source
- `v1` = latest compatible v1 line
- full commit SHA = immutable production/reference pin

Breaking workflow interfaces require a new major version rather than silently changing existing callers.

## Repository maintenance

- Pull requests use a standard checklist.
- Issues use structured bug/feature forms.
- `CODEOWNERS` defines ownership.
- The repository uses the MIT license.
- Dependabot groups GitHub Action updates into a weekly maintenance PR.
- Repository workflow changes run through blocking security checks.

## Design rules

1. Automations should be conservative by default.
2. Never auto-merge dependency upgrades.
3. Major dependency upgrades stay manual.
4. Validate before creating a pull request.
5. Give workflows only the permissions they need.
6. Centralize shared logic instead of copying it into every project.
7. Prefer built-in GitHub/CLI functionality over unnecessary third-party actions.
8. Adopt new security checks in observation mode before making them blocking in older projects.
