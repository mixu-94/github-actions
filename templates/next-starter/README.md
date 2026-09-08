# Next Starter

A small, opinionated Next.js starter for new projects.

It intentionally includes engineering defaults without forcing application architecture decisions too early.

## Included

- Next.js 16.3.4
- React 19.2.8
- TypeScript
- ESLint
- App Router
- shared CI via `mixu-94/github-actions`
- shared GitHub Actions security checks
- conservative dependency-update workflow
- pull-request template
- security policy

## First setup

```bash
npm install
npm run dev
```

Commit the generated `package-lock.json` immediately after the first install. Once a lockfile exists, change the shared CI caller from `install-command: npm install` to `install-command: npm ci` for fully reproducible CI installs.

The monthly dependency updater already expects a committed lockfile, so do the first `npm install` before running it.

## Scripts

```bash
npm run dev
npm run build
npm run start
npm run lint
npm run check-types
```

## Why the GitHub workflows are tiny

The project only declares *when* automation runs and which options it wants. The implementation lives in `mixu-94/github-actions`.

That means a new project gets the same CI/security behaviour without copying large workflow files around.

The reusable workflows are pinned to an immutable commit so the starter does not silently change when the central repository changes.

## After creating a real project

1. Replace the project name and README content.
2. Run `npm install` and commit `package-lock.json`.
3. Switch CI installation to `npm ci`.
4. Enable GitHub Dependency Graph / Dependabot security features where available.
5. Add tests when the project has behaviour worth testing, then enable the CI test command.
6. Add deployment/release workflows only when the project actually needs them.
