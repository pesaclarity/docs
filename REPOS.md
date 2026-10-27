# Repositories

The `pesaclarity` GitHub organisation holds one repository per licence boundary from the spec's open-source section. All exist now; each is built in the phase shown. Codenames are Swahili words used for package and module names.

| Repo | Codename | What | Licence | Built in |
| --- | --- | --- | --- | --- |
| [`.github`](https://github.com/pesaclarity/.github) | | Org profile, contributing guide, code of conduct, security policy, issue templates | CC BY 4.0 | Phase 0 |
| [`docs`](https://github.com/pesaclarity/docs) | | This repo: spec, brand, decisions | CC BY 4.0 (text), trademarks excluded | Phase 0 |
| [`ujumbe-parsers`](https://github.com/pesaclarity/ujumbe-parsers) | ujumbe (message) | SMS parser packs per institution, Dart engine, synthetic fixtures, pack signing | Apache-2.0 | 1. Parsers and engine (weeks 3–6) |
| [`pesaclarity-app`](https://github.com/pesaclarity/pesaclarity-app) | | Flutter Android app: screens, Drift + SQLCipher ledger, budget, debts, FHI, backups. Play Store and F-Droid flavours | AGPL-3.0 + commercial on request | 1–2 (weeks 3–11) |
| [`pesaclarity-web`](https://github.com/pesaclarity/pesaclarity-web) | dirisha (window) | Website: what Pesa Clarity is, principles, privacy, open source, calls to action | AGPL-3.0 (code), CC BY 4.0 (text) | 2, before Play review |
| [`pesaclarity-backend`](https://github.com/pesaclarity/pesaclarity-backend) | ngome (fortress) | Auth, encrypted backup vault, federated-learning aggregator. Docker, GitHub Actions | AGPL-3.0 + commercial on request | 2–3 (weeks 7–14) |
| [`pesaclarity-ml`](https://github.com/pesaclarity/pesaclarity-ml) | akili (intelligence) | Categoriser, DP-SGD config, TFLite export, MLflow tracking, release audits | AGPL-3.0 (code); weights under a separate model licence | 3. Beta (weeks 12–14) |

`pesaclarity-app` depends on `ujumbe-parsers` as the Dart package `ujumbe`, so packs version and release independently of the app.

## Rules for every repo

- Synthetic or fully anonymised test data only. Never commit real SMS, statements or phone numbers.
- No secrets in any repo. App signing, pack signing and server keys live in a hardware key or secrets manager.
- `main` is protected: pull requests, passing CI, no force-push.
- Pull request titles follow Conventional Commits; they set versions and changelogs.
- `pesaclarity-app`, `pesaclarity-backend` and `ujumbe-parsers` publish from `main`, so changes go to `dev` and reach `main` after CI and QA ([0008](decisions/0008-dev-branch-and-release-please.md)).
- Every repo links back here for the spec and brand.
