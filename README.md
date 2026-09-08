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

Shared CI for Node.js and Next.js-style projects.

It can run:

- clean npm installs
- builds across multiple Node.js versions
- Conventional Commit validation
- linting
- TypeScript checks
- unit tests
- Codecov uploads
- Playwright installation
- Storybook tests
- end-to-end tests
- test-result artifact uploads

Most commands are configurable inputs. This keeps the workflow reusable instead of coupling it to one specific app.

Think of this workflow as the common project TÜV: every repository can be checked against the same basic quality rules.

### `release-node.yml`

Shared semantic-release workflow.

It performs a clean checkout, installs dependencies and runs the configured release command with only the permissions needed for GitHub releases, issues and pull requests.

The caller decides when a release should happen. For example, a project can trigger it after CI completes and explicitly require `github.event.workflow_run.conclusion == 'success'`.

This separation is useful because "how to release" stays centralized while "when to release" remains a project decision.

### `security.yml`

Shared security checks.

Current checks:

- GitHub Dependency Review for dependency changes in pull requests
- `zizmor` auditing for GitHub Actions workflow security

The default mode is non-blocking. Findings are visible but do not immediately prevent merges. This is useful for adopting security checks in older/reference projects without turning historical warnings into sudden build failures.

Callers can later set `blocking: true` after findings have been reviewed.

### `update-npm-dependencies.yml`

Safely refreshes npm dependencies and opens a pull request.

Default behaviour:

- patch updates only
- Node.js 20.x
- keeps `package-lock.json` instead of deleting it
- verifies the current and updated dependency tree with clean installs
- runs `check-types` and `test` when those npm scripts exist
- build verification is opt-in because builds often require project-specific environment variables
- opens a pull request only after validation succeeds
- never auto-merges dependency changes

Minor updates can be enabled explicitly by the caller. Major updates are intentionally not supported by the automated workflow and should be reviewed manually.

## Composite actions

### `actions/create-pr`

Small in-house action for the common "commit generated changes and open/update a pull request" use case.

It uses Git and GitHub CLI (`gh`) instead of a large third-party implementation:

1. detect changes
2. create/reset a dedicated automation branch
3. commit the changes
4. push the automation branch
5. create a pull request, or update the existing one

The automation branch is deliberately treated as disposable state and may be force-updated by this action. It should never be used for manual development work.

Why own this one? The use case is simple enough to understand and maintain ourselves, which reduces third-party code in a workflow that needs write permissions.

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
    uses: mixu-94/github-actions/.github/workflows/ci-node.yml@main
    with:
      build-node-versions: '["20.x","22.x"]'
      validate-commits: true
      install-playwright: true
      storybook-command: npm run test-storybook:ci
      e2e-command: npx percy exec -- npm run test:e2e
```

## Example: dependency updater caller

The schedule belongs to the consuming repository:

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

Optional inputs:

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

## Versioning

During initial development the project callers use `@main` so improvements are easy to iterate on.

Once these workflows have settled, callers should move to a stable version tag such as `@v1` or, for maximum supply-chain safety, a full commit SHA.

## Third-party actions

We use upstream actions when they solve genuinely complicated problems better than a small local implementation.

Security-sensitive third-party actions should preferably be pinned to full commit SHAs. Dependabot keeps GitHub Actions references up to date from this central repository.

If we fork or copy code from another action, its license and attribution must be checked first.

## Design rules

1. Automations should be conservative by default.
2. Never auto-merge dependency upgrades.
3. Major dependency upgrades stay manual.
4. Validate before creating a pull request.
5. Give workflows only the permissions they need.
6. Centralize shared logic instead of copying it into every project.
7. Prefer built-in GitHub/CLI functionality over unnecessary third-party actions.
8. Adopt new security checks in observation mode before making them blocking.
