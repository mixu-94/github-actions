# Security Policy

## Reporting a vulnerability

Please do not open a public issue for a suspected vulnerability that could expose secrets, allow unintended repository writes, or affect repositories consuming these workflows.

Use GitHub's private vulnerability reporting feature when it is available for this repository. If private reporting is not available, contact the repository owner privately through an established contact channel.

When reporting, include:

- the affected workflow or action
- the relevant commit or version
- what an attacker could do
- minimal reproduction steps
- any suggested mitigation, if known

## Scope

Security-sensitive areas include:

- token and permission handling
- third-party GitHub Actions
- branch or pull-request automation
- release workflows
- command or input injection paths
- reusable workflow trust boundaries

## Supported versions

The latest maintained major version receives security fixes. Consumers pinned to an older immutable commit should update after reviewing the corresponding security change.
