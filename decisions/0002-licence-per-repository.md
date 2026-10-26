# 0002. One repository per licence boundary

Date: 2026-10-26 · Status: Accepted

## Context

The spec licenses parts differently: the app and server under AGPL-3.0 with a commercial option, parser packs under Apache-2.0, docs under CC BY 4.0, model weights under a separate licence. Mixing licences in one repository makes it unclear what a contributor or reuser gets.

## Decision

Each licence boundary is its own repository with a single `LICENSE` at the root: `.github` and `docs` (CC BY 4.0), `ujumbe-parsers` (Apache-2.0), `pesaclarity-app`, `pesaclarity-backend`, `pesaclarity-ml` and `pesaclarity-web` (AGPL-3.0). Website text is CC BY 4.0. Model weights are never committed.

## Consequences

- Banks and fintechs can adopt parser packs without AGPL obligations.
- The commercial licence for the AGPL repos needs a contributor licence agreement before outside code is merged.
- Cross-repo changes take more than one pull request.
