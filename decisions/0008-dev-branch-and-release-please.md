# 0008. dev branch and release-please

Date: 2026-10-27 · Status: Accepted

## Context

In `pesaclarity-app`, `pesaclarity-backend` and `ujumbe-parsers`, `main` is published: it releases to Google Play, deploys the server, or signs the packs phones download. Changes merged straight to `main` reach people before anyone has tried them together. Versions were bumped by hand and changelogs written by hand, so they drifted.

## Decision

Pull requests go to `dev`, titled in Conventional Commits form and squash-merged. `dev` reaches `main` only through a pull request that passes CI and a QA approval (the `qa` environment), merged with a merge commit. release-please on `main` sets the next SemVer version from the titles, writes the changelog and tags `vX.Y.Z`; the tag runs the release (Play upload or pack signing) and the release is merged back into `dev`.

## Consequences

- `main` only moves through tested, approved batches; `dev` builds give QA a test APK for every push.
- Versions and changelogs follow from pull request titles; a `PR title` check enforces the form.
- dev-to-main merges must keep the individual commits (merge commit, not squash), or release-please sees one untyped change.
- Branch protection on `main`, required checks and environment reviewers need GitHub Pro or public repositories; until then the release gate and QA checks show red but cannot block a merge, so following them is by agreement.
- Other repos keep merging to `main`.

## Amendment, 2026-11-05

`pesaclarity-ml` follows the same flow: its `main` is what model releases are trained and exported from, so it gets a `dev` branch, the release gate and release-please too. `docs`, `pesaclarity-web` and `.github` keep merging to `main`.

## Amendment, 2026-11-16

Checks now run on the developer's machine, not on GitHub. `pesaclarity-app`, `ujumbe-parsers` and the website each have `tool/check`; `tool/setup-hooks` turns on a pre-push hook that runs it, and in the app and parser repos also checks that each pushed commit has a Conventional Commits title. That hook replaces the `PR title` check, and where this record says "passes CI", read "`tool/check` passed before the push". GitHub Actions only builds, releases and deploys. `pesaclarity-web` is now `pesaclarity.github.io`.
