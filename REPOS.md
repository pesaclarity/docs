# Repositories

The `pesaclarity` GitHub organisation holds one repository per licence boundary from the spec's open-source section. Create each when its roadmap phase starts, not before.

| Repo | What | Licence | Create in phase |
| --- | --- | --- | --- |
| [`.github`](https://github.com/pesaclarity/.github) | Org profile page, default contributing guide, code of conduct, security policy | CC BY 4.0 | Now |
| [`docs`](https://github.com/pesaclarity/docs) | This repo: spec, brand, decisions | CC BY 4.0 (text), trademarks excluded | Now |
| `parser-packs` | SMS and statement parser packs per institution, synthetic test fixtures, pack signing tooling | Apache-2.0 | 1. Parsers and engine (weeks 3–6) |
| `app` | Flutter Android app: on-device engine, Drift + SQLCipher, UI. Play Store and F-Droid flavours | AGPL-3.0 + commercial on request | 1–2 (weeks 3–11) |
| `server` | Auth, encrypted backup vault, federated-learning aggregator. Docker, GitHub Actions | AGPL-3.0 + commercial on request | 2–3 (weeks 7–14) |
| `models` | Federated training code, DP-SGD config, MLflow tracking, release audits. Weights under a separate model licence | AGPL-3.0 (code) | 3. Beta (weeks 12–14) |
| `website` | Landing page, privacy policy, terms | CC BY 4.0 | Before Play review |

## Rules for every repo

- Synthetic or fully anonymised test data only. Never commit real SMS, statements or phone numbers.
- No secrets in any repo. App signing, pack signing and server keys live in a hardware key or secrets manager.
- `main` is protected: pull requests, passing CI, no force-push.
- Every repo links back here for the spec and brand.
