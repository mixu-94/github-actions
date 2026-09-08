# Contributing

Thanks for improving this automation toolbox.

## Principles

Changes should keep the workflows understandable, conservative, and reusable.

- Prefer built-in GitHub and CLI capabilities over unnecessary dependencies.
- Pin third-party actions to immutable commit SHAs.
- Keep permissions as small as possible.
- Avoid automatic major dependency upgrades or auto-merges by default.
- Preserve backwards compatibility within a major version when practical.
- Explain security-sensitive behaviour in comments or documentation.

## Pull requests

Please keep pull requests focused and include:

- what changed
- why the change is useful
- how it was validated
- whether callers need to change anything
- whether permissions, secrets, or trust boundaries changed

Breaking changes should be clearly marked and should normally target the next major version.

## Testing workflow changes

For reusable workflows, prefer validating changes from a caller workflow or a draft pull request before moving the stable major version reference.

For composite actions, test both the normal path and the no-op path where no changes are present.
