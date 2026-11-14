# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A composite GitHub Action (`mhvelplund/notify-failure`) that sends a short ntfy.sh push notification when a workflow run
fails. There is no build step and no test suite. The whole action is `action.yml` plus `notify.sh`.

## Architecture

- `action.yml` declares the inputs and runs one bash step that calls `"$GITHUB_ACTION_PATH/notify.sh"`. **Every value is
  passed through `env:`, never through `${{ }}` inside `run:`**, so the ntfy topic is never part of the rendered script
  text. Keep this pattern when adding inputs.
- `notify.sh` reads only environment variables:
  1. If `NTFY_TOPIC` is empty, it logs a warning and exits 0. A missing secret must never fail the caller's workflow.
  2. It calls `gh api` (with `GH_TOKEN`, which needs `actions: read` and `checks: read`) to find the first failed job
     and step of the current run attempt. It then picks the first failure annotation that says more than "Process
     completed with exit code N", and falls back to the exit code.
  3. Every lookup is best effort. API errors fall back to generic values so the alert is always sent. The script uses
     `set -uo pipefail` on purpose, without `-e`.
  4. It POSTs to `https://ntfy.sh/$NTFY_TOPIC` with ntfy headers (`Title`, `Priority`, `Tags`, `Click`, `Actions`).
- The topic is a secret: never echo it, and never add `set -x` or `curl -v` to `notify.sh`.
- `.gitattributes` forces LF line endings. The script must keep LF endings to run on Linux runners (development happens
  on Windows).

## Releases and versioning

- Releases are automated by `.github/workflows/release.yml` on every push to `main`. It uses commitizen (config in
  `.cz.toml`, where the version is stored and tags are `v$version`).
- **Commit messages must be Conventional Commits.** `fix:` and `feat:` commits (and breaking changes) cut a `vX.Y.Z`
  tag, a GitHub release, and a `CHANGELOG.md` entry. `ci:`, `docs:`, `build:` and `chore:` commits don't release
  anything.
- The workflow commits as the `commitizen-ci` GitHub App (client id in the workflow, private key in the repo secret
  `COMMITIZEN_CI_PRIVATE_KEY`). It skips commits that start with `bump:` so its own bump push doesn't loop.
- After each release it force-moves the floating `vX` and `vX.Y` tags to the new release. Users reference `@v1` or
  `@v1.2`, so never delete or retag these by hand without moving them consistently.

## Commands

- Lint the workflow: `mise exec actionlint@latest -- actionlint`
- Check the commitizen config or preview a bump: `uvx --from commitizen cz version --project` and
  `uvx --from commitizen cz bump --dry-run --yes`
- `gh` is installed through mise with no global default version, so run it as `mise exec gh@2.103.2 -- gh ...`.
