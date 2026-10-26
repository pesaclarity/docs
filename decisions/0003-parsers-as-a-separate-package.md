# 0003. Parser packs as a separate Dart package

Date: 2026-10-26 · Status: Accepted

## Context

Institutions change their message wording without notice. Packs must be fixable within days, accept community contributions, and stay under Apache-2.0 while the app is AGPL-3.0.

## Decision

`ujumbe-parsers` publishes the Dart package `ujumbe`: the engine plus YAML packs with synthetic fixtures. `pesaclarity-app` depends on it by version. Packs also ship to phones as Ed25519-signed files, so a fix does not need an app release.

## Consequences

- A pack releases only when all its fixtures pass.
- Pack signing keys live outside the repository.
- The app pins an engine version and accepts any signed pack compatible with it.
