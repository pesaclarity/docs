# 0024. Counting installs and crash reports without tracking

Date: 2026-11-18 · Status: Proposed · Builds on [0004](0004-eu-hosting-and-email-sign-in.md), [0006](0006-fdroid-build-without-prebuilt-libraries.md) and [0019](0019-on-device-ml-without-google.md)

This is a plan. Effort figures are estimates, not measurements. Mockups (not built) go with the pull request and need the owner's approval before anything is built.

## Context

The owner asked for user counts, devices, demographics, crash reports and a dashboard only the owner can open. What exists today:

- **Downloads.** The `Download stats` workflow in `pesaclarity/.github` records GitHub's public download counters for every `pesaclarity-app` release file once a day (00:17 Nairobi) and appends them to `stats/downloads.csv` on the `stats` branch. It counts files, not phones, and knows nothing about Play, F-Droid or iPhone installs.
- **The update check.** The app calls `GET /v1/app-status?version=&build=&platform=` on ngome when it opens, at most once a day (`appStatusInterval`), and again when its version changes. `build` is `play`, `foss`, `ios` or `website`. It sends no account, token or id (`HttpAppStatusSource`). The answer is signed with Ed25519 and pinned in the app. The server stores nothing about the caller: its access log records only the route template, status and duration, and Uvicorn's access log is off (`ngome.core.logging`). Google Cloud still keeps Cloud Run's request logs (address, time, URL), as privacy policy 2.2 says. Notices are created with `POST /v1/admin/app-notices` and the admin bearer token (`NGOME_ADMIN_TOKEN`); unset, the admin routes return 404.
- **Crash reports.** Policy 2.2 and the spec list "Anonymous crash reports: off unless you opt in", and `ConsentKind.crashReports` (default off) is a switch in Privacy, but nothing captures or sends a report: the app sets no `FlutterError.onError` or `PlatformDispatcher.onError` handler. Release builds are obfuscated with `--split-debug-info`, and the symbol files are kept per release.
- **Sign-in on the server.** Email codes, and Google ID tokens from the Play app's client ids (`NGOME_GOOGLE_CLIENT_IDS`), keeping only a keyed hash of the subject. There is no web sign-in and no admin user, only the admin token.
- **Federated learning.** `ngome.fl.server` builds a Flower ServerApp (FedAvg under SecAgg+, client-side clipping, smallest cohort 100). It persists no rounds and no privacy spend yet (a `TODO(fl)` in the code), so there is nothing to show about it.
- **Promises.** The spec says "No analytics" with one opt-in exception for crash reports, and "No third-party SDKs or trackers". The policy promises "no ads, no analytics and no third-party trackers", and Privacy's first promise card says "We store only your sign-in details and backups we cannot open."
- **F-Droid.** F-Droid's Anti-Features page names two examples of the *Tracking* anti-feature: sending crash reports, and checking for updates, without the user's knowledge or permission. It does not apply when the feature is opt-in and off by default, needs informed consent with a GDPR-like privacy policy, and avoids personal data as far as possible.
- **The law.** The Kenya Data Protection Act 2019 requires data to be adequate, relevant and limited to what is necessary (section 25), and a data protection impact assessment where processing is likely to be high risk (section 31). The spec already plans one for federated learning.

## Decision

Count active installs with a monthly random number on the update check, collect no demographics, send crash reports only when the person says yes after seeing them, and show all of it on a read-only dashboard that ngome serves to the owner's passkey.

### 1. Active installs, without identifiers

**On the phone.** The app makes a counting token: 128 random bits from `Random.secure()`, encoded as 22 base64url characters. It is never derived from anything about the phone, the person or an account. It is made on first use in a calendar month (UTC) and replaced on the first check-in after the 1st of the next month. It lives in the app's main settings, outside every space and outside backups, so a restored phone does not reuse it.

- It is sent only on `GET /v1/app-status`, in a request header (`Count-Token`), never in the URL, so Cloud Run's request logs, which record the URL and address, never hold it. It is never sent with sign-in, backups, model updates or crash reports.
- It is sent only after the person has seen policy 2.3 (a material change, so the existing policy review screen shows it first), and only while "Count this phone in monthly totals" is on in Privacy. Switching it off deletes the token; the update check carries on without it.
- What it counts is installs, not people: a reinstall, or two phones, count twice that month.

**On the server** (a new `ngome.counting` module):

| Table | Holds | Never holds |
| --- | --- | --- |
| `count_keys` | One random 32-byte key per open period (each day and each month) | Anything about a phone |
| `count_seen` | `(period, digest)`: the first 16 bytes of HMAC-SHA-256 of the token under that period's key | Version, build, country, address or time |
| `count_totals` | `(period, dimension, value, n)`, one dimension per row: `all`, `build`, `version`, and `country` for months only | Any token or digest |
| `notice_reach` | `(notice_id, day, n)`: how many check-ins got each notice | Who got it |

On a check-in with a well-formed token the server, in one transaction, inserts the digest for today and for this month. Only when an insert is new does it add one to that period's totals for `all`, the build, the version and, for the month, the country. A day's or month's totals therefore count each token once, under the version the phone had when first seen in that period. Because the day and month keys differ, a day digest cannot be matched to a month digest.

- **Exact set, not a sketch.** At the expected scale (thousands of phones) an exact set is a few hundred kilobytes a month and gives exact counts. HyperLogLog would hide the tokens a little better but adds error; reconsider it above a million rows a month.
- **Deleted when the period closes.** The daily purge job (`ngome-purge`) deletes the day sets and keys older than yesterday, and the month's set and key on the 2nd of the next month (a day's grace for phone clocks). After that, nothing links a total to a phone, and a token from a past month cannot be recognised.
- **Country** comes from the request's address at request time (Cloud Run puts the client address in `X-Forwarded-For`; `FORWARDED_ALLOW_IPS` is already `*`), looked up in an offline country database held in memory, and then dropped. The address is never stored, logged or hashed. The database is **DB-IP IP to Country Lite** (MMDB format, CC BY 4.0, attribution "IP Geolocation by DB-IP" with a link to db-ip.com on the dashboard and in the policy), read with the `maxminddb` reader (Apache-2.0) and refreshed monthly in the image build. MaxMind GeoLite2 is not used: it needs an account and licence key and comes under MaxMind's own end-user licence rather than an open one. Addresses the database cannot place count as "Unknown".
- **Small numbers are hidden.** The dashboard shows any count under 10 as "<10", and also hides a second cell where the total would reveal a hidden one. Country is offered by month only, one dimension at a time (no country by version or by build), so few cells are small. Stored totals are exact; suppression is applied on display.

No device ids, IMEI, Android ID, advertising id, account, phone model, screen size, locale, time zone or fingerprinting goes into counting.

### 2. No demographics

Age, gender, precise location, income and any other description of the person are out, now and in this design's future versions:

- The privacy promise is that money and personal details stay on the phone; asking for them, or inferring them, breaks it.
- The Act's minimisation principle: no use we have needs them. The app is for people aged 18 and over, which the policy already says without asking anyone's age.
- They would earn F-Droid's *Tracking* anti-feature and invite the kind of profiling the spec rules out ("no credit scoring").
- Country is the coarsest place we can know without asking, and it is never stored with anything about a phone.

Product questions that need more than counts go to interviews and opt-in feedback, as the spec already says.

### 3. Crash reports, opt-in per crash

**What is captured.** Uncaught Dart errors (`FlutterError.onError` for fatal framework errors, `PlatformDispatcher.instance.onError`, and errors from the app's own isolates). The app saves at most three pending reports to a file and, on the next launch, shows one sheet. Native crashes in Android or iOS code are not covered at first (open question 7).

**What is sent**, and nothing else. The report is built field by field from fixed sources (an allow-list), never by cleaning up a string:

| Field | Example | Source |
| --- | --- | --- |
| `format` | `1` | constant |
| `app`, `build_number`, `build` | `0.5.2`, `412`, `play` | the build's own values |
| `os` | `Android 13` | the Android API level (or iOS major version), not `Platform.operatingSystemVersion`, which can carry the build fingerprint |
| `cpu` | `arm64` | `Abi.current()` |
| `error` | `TypeError` | the first type in a fixed list the error `is` (`TypeError`, `StateError`, `RangeError`, `ArgumentError`, `FormatException`, `SqliteException`, `TimeoutException`, `FileSystemException`, `PlatformException`), else `Other`. Never `toString()` or the message, which can hold message text, amounts or names |
| `build_id` | `b6f1c4e09a2d7f35` | the ELF build id printed in the stack's header |
| `stack` | `["#00 pc 0x2f4b3c", ...]` | up to 64 frames, each reduced to its offset in the app's instructions; in an unobfuscated build, frames from `package:pesaclarity/`, `package:` and `dart:` only. Any line that does not match is dropped |

No amounts, messages, names, phone numbers, contacts, file paths, screen contents, breadcrumbs, locale, time zone, device model or time of day. The server records the day it received a report, not the time. The **scrubber** is this builder plus tests: a test plants amounts, `+2547…` numbers, emails, names, message text and `/data/user/0/…` paths in error messages and stack lines and checks that none reach the payload; a golden test fixes the payload's shape; a size cap (16 KB) rejects the rest. The server validates the same schema again (Pydantic, `extra="forbid"`, a regular expression per field) and refuses anything else.

**The sheet** (mockups): "Pesa Clarity closed last time", what a report is for, **See what will be sent** expanding to the exact JSON that would be sent, **Send report** and **Don't send**, and **Do the same next time**, which turns the choice into Always send or Never send. Privacy gets a **Crash reports** row with **Ask each time** (default), **Always send** and **Never send**. The existing on/off `crashReports` consent becomes this three-way choice: off becomes Ask each time, on becomes Always send. First run no longer asks about crash reports, because the first crash does.

**Transport and storage: options compared.**

| | (a) ngome: `POST /v1/crash-reports` into Postgres | (b) Self-hosted GlitchTip (MIT) |
| --- | --- | --- |
| Runs on | The existing Cloud Run service and Neon database | A second service: a Django web and worker process, Postgres and optionally Valkey; about 0.5 to 1 GB of memory by third-party guides, beyond our free tiers |
| App side | About 300 lines of Dart we own; payload exactly as above | A Sentry SDK. `sentry_flutter` brings Sentry's native Android and iOS SDKs as prebuilt artefacts, the problem 0006 and 0019 removed; pure-Dart `sentry` avoids that but sends Sentry's event format, whose device and context fields we would have to strip and keep stripping |
| Sign-in | The dashboard's passkey | GlitchTip's own accounts, a second admin to secure |
| Grouping and UI | Ours, simple (error type plus first frames in our code) | Mature |
| Retention | The purge job we already run | GlitchTip's own settings |

**Recommendation: (a).** One service, one sign-in, a payload we can show the person byte for byte, and nothing prebuilt in the F-Droid build. GlitchTip stays the fallback if our grouping proves too weak. Firebase Crashlytics and every other Google or third-party crash service are out.

**On the server** (`ngome.crashes`): the endpoint needs no account and sets no cookie; it is not linked to the counting token. A report is stored with its received day, a fingerprint (error type plus the first three decoded frames in our code) and its group. **Symbolication**: the release workflows already keep `--split-debug-info` symbols; they upload each release's symbol files to object storage, and a small Cloud Run job with the Dart SDK decodes new reports (with `package:native_stack_traces`, BSD-3-Clause) so the API image stays small. F-Droid builds decode only if F-Droid ships our reproducible build (the build id then matches); otherwise the dashboard counts them as not decoded. Limits: 16 KB per report, a global daily cap (setting, default 2,000), at most three per launch from the app.

**F-Droid.** Per-crash asking is opt-in and off until the person says yes, with the exact content shown, so it meets F-Droid's condition for crash reporting; Always send is the person's own choice. No anti-feature is expected for crash reports. Counting is different: it rides on an update check that runs without asking, which is one of F-Droid's named *Tracking* examples. So the F-Droid build ships with counting **off** until the person ticks it (recommended; open question 1), and whether the update check itself needs asking for in that build is open question 2.

### 4. The admin dashboard

Served by ngome under `/admin`, not GitHub Pages, so it can read the database and needs no API token in a browser. Server-rendered HTML (Jinja2, BSD-3-Clause), charts drawn as inline SVG on the server, no third-party scripts, fonts or requests. One first-party script runs the passkey ceremony. Headers: `Content-Security-Policy: default-src 'self'; frame-ancestors 'none'`, `Cache-Control: no-store`, `Referrer-Policy: no-referrer`. Read-only at first: notices are still sent with the admin token.

**Sign-in: passkeys (WebAuthn) with an allow-list**, using `webauthn` (py_webauthn, BSD-3-Clause). Google sign-in was considered, since ngome already verifies Google ID tokens, but a web sign-in would need a new web OAuth client and Google's sign-in script or redirect, and would make Google the gatekeeper to the owner's data; passkeys need neither.

- Enrolment: a terminal command with the admin token mints a single-use enrolment link valid for 10 minutes; the owner registers a passkey (phone screen lock or security key). Register two, so one can be lost. Only credential ids on the allow-list (`admin_passkeys` table: credential id, public key, sign counter, a label) can sign in.
- Session: a random id in a `__Host-` cookie (`Secure`, `HttpOnly`, `SameSite=Strict`), 12 hours, stored server-side; sign-out deletes it. State-changing requests also check `Origin`.
- Lost every passkey: enrol again with the admin token. The admin token never reaches a browser.

**Pages** (mockups, desktop and phone, light and dark):

- **Sign in**: one button, no password, no email code.
- **Overview**: monthly active installs (lead card), yesterday's daily installs and the 7-day average, GitHub downloads in the last 30 days (read daily from `stats/downloads.csv` on the `stats` branch of `pesaclarity/.github`, public, no token), crash reports in 7 days; daily installs over 30 days; versions and builds this month; notice reach; downloads by file; a federated-learning card that says "not running yet" until the aggregator records rounds.
- **Installs**: by country for a month, with "<10" cells, a note on suppression and on what is stored, and the flood check (below).
- **Crashes**: groups with error, first frame, versions, count, trend, first and last seen; a group page with decoded stack, builds, Android versions, CPU, reports per day, and one report exactly as received.

### 5. Policy 2.3, the DPIA, retention and abuse

**Privacy policy 2.3** (app `assets/policy/privacy_policy.md` and `.json`, website `src/pages/privacy.astro`; wording to be reviewed by the Kenyan data protection lawyer the spec names):

In "The short version", "Our server does four narrow jobs" becomes "Our server does five narrow jobs".

"Update check" becomes:

> Once a day, with or without an account, the app asks our server whether a newer version or a notice is waiting, such as a fix for a bank that changed its messages. The request carries the app's version, which build it is (Play, F-Droid, iPhone or website) and the platform, and, unless you switch it off, the counting number described below. It carries no account, device or phone id. The answer is signed, so the app shows only notices that really come from us. The app asks you to update before you carry on only when a version is known to get amounts wrong.

New, after it, "Monthly count":

> So we know how many phones use Pesa Clarity, the update check carries a random number that your phone makes each month and replaces on the 1st of the next. It is not made from anything about you or your phone, and it is never sent with anything else. Our server uses it only to count each phone once a day and once a month, adding one to totals by app version, build and, for the month, country. It keeps those totals, which name no one. It keeps a scrambled copy of the number only until the day or month is over, then deletes it. We work out the country from your internet address while answering, with a database kept on our server (IP Geolocation by DB-IP), and never store or log the address. We never look at a total under 10. To stop, switch off "Count this phone in monthly totals" in Privacy; the update check carries on without the number. In the F-Droid build it is off until you switch it on.

New, "Crash reports":

> If the app closes because of a fault, it asks on the next start whether to send us a crash report, and shows you exactly what it would send. A report holds the app's version and build, the phone's system version (such as Android 13) and chip type (such as arm64), the kind of error, and where in the app's own code it happened, as numbers. It never holds amounts, messages, names, phone numbers, contacts or anything you typed, and it carries no account, device or install id. It is not linked to the monthly count. We keep reports for 90 days, then only how many there were and when. In Privacy you can choose Ask each time, Always send or Never send.

"Who runs our server", the Google Cloud line gains: "These logs never include the counting number or crash reports' contents."

"Your choices" becomes:

> - Improve the app (federated updates): on by default.
> - Contribute to licensed models: off unless you opt in.
> - Count this phone in monthly totals: on by default; off by default in the F-Droid build.
> - Crash reports: the app asks each time, unless you choose Always send or Never send.
>
> Pesa Clarity has no ads and no third-party trackers, and no analytics beyond the monthly count above.

Changes entry for 2.3: "The update check can carry a random number, new each month, so we can count how many phones use the app; you can switch it off. Crash reports are sent only when you say yes, and you can see exactly what is sent." The Privacy promise "We store only your sign-in details and backups we cannot open." becomes "We store only your sign-in details, backups we cannot open and counts that name no one."

**DPIA note.** Add counting and crash reports to the impact assessment the spec plans for federated learning. Counting: a pseudonymous token, personal data while it can still be linked within its period, minimised to one random value, deleted at period end; outputs are anonymous aggregates. Legal basis: legitimate interests (knowing whether the app is used and which versions need support), with an easy objection (the switch). Crash reports: consent, given per report or by choosing Always send, with the content shown first. Risks assessed: linking a token to an address (the address is never stored with it; Google's request logs never see the header), small cells revealing a person in a small country (suppression, country by month only), a crash report leaking personal data (allow-list builder, server schema, preview), and the dashboard being broken into (passkeys, aggregates only). Most of this is unlikely to be high risk, but writing it down is cheap and the regulator may ask.

**Retention**

| Data | Where | Kept |
| --- | --- | --- |
| Counting token | the phone | until the month ends or the switch goes off |
| Internet address | server memory during the request | never stored or logged by ngome |
| Day digests and day keys | Postgres | until the day after |
| Month digests and month key | Postgres | until the 2nd of the next month |
| Daily totals | Postgres | 13 months |
| Monthly totals | Postgres | kept (anonymous) |
| Notice reach | Postgres | 13 months |
| Crash reports | Postgres | 90 days |
| Crash groups (counts, dates, decoded first frames, no reports) | Postgres | until a year after last seen |
| Release symbol files | object storage | 2 years per release |
| Admin passkeys; admin sessions | Postgres | until removed; 12 hours |
| Cloud Run request logs (address, time, URL; no headers) | Google Cloud Logging | its default retention, unless excluded (open question 3) |

**Abuse.** Anyone can make up tokens and inflate the counts; we say so, and keep the harm small rather than pretend to stop it:

- Tokens must be 22 base64url characters; anything else is ignored and the check-in still answered.
- A ceiling on new digests per hour (setting, default 600 per instance); above it, extra tokens are not counted and the hour is marked as a flood on the dashboard. A cap on rows per period protects the database. Cloud Run's maximum instances bounds the bill.
- No per-address limits: they would need the address, or a hash of it, kept somewhere. Proof of work was considered and rejected: cheap for an attacker with a GPU, costly on a low-end phone. Play Integrity or App Attest could prove a genuine app, but not for F-Droid or website builds, and it would add Google to every check-in.
- The counts drive nothing automatically (no rollout or notice depends on them), so a flood skews a chart and nothing else.
- Crash reports: schema, size cap and daily cap; a junk report lands in its own group.

**How anyone can check.** All of it is in open repositories: the header in `HttpAppStatusSource`, the token's making and deletion, the crash report builder and its tests in `pesaclarity-app`; `ngome.counting`, `ngome.crashes`, the purge job and the dashboard in `pesaclarity-backend`. Reproducible builds tie the APK to the source. The sheet shows the exact bytes before sending.

### 6. Stages

| Stage | Work | Effort (estimate) |
| --- | --- | --- |
| 0 | Owner answers the open questions and approves the mockups; lawyer reviews the 2.3 wording | owner time |
| 1 | Server counting: tables and migration, the check-in path, DB-IP in the image build with attribution, purge, flood ceiling, tests | 3 days |
| 2 | App token, Privacy switch (F-Droid default off), policy 2.3 in the app and on the website, backend README and spec updates | 2 days |
| 3 | Dashboard v1: passkey enrolment and sign-in, sessions, overview, installs, downloads from the CSV, notice reach | 4 to 5 days |
| 4 | Crash reports: app capture, builder and scrubber tests, sheet and settings (2.3 already covers them); `POST /v1/crash-reports`, storage, purge; symbol upload and the decode job; dashboard crash pages | 10 to 11 days |
| 5 | Federated-learning card, once the aggregator stores rounds and privacy spend | with the FL work |

Stages 1 and 2 ship together (no token is sent before the server counts it, and no count starts before 2.3 is shown). Stage 4 can ship without stage 3's crash pages only if reports are read with SQL meanwhile.

## Consequences

- The owner gets monthly and daily active installs by version, build and country, update-notice reach, downloads and crash groups, without any id that outlives a month, any address on our side, or any third party.
- "No analytics" is no longer literally true; the policy, the spec's privacy section and server table, the backend README's "App status … Anything about who asked" row and the Privacy promise card must say what is counted. Policy 2.3 is a material change and is shown before counting starts.
- Counts are estimates: reinstalls, several phones, counting switched off, offline phones and made-up tokens all move them. F-Droid users count only when they opt in, so F-Droid is under-counted.
- Monthly figures by version use the version a phone had when first seen that month.
- We own a small crash pipeline (builder, schema, grouping, decode job) instead of a mature tool; every new error type to name means a line in the allow-list.
- Dart errors only at first; native crashes are not seen.
- The dashboard adds web sign-in to a server that had none; passkeys keep it to the owner and out of Google's hands.

## Open questions for the owner

1. **Counting default.** Recommended: on, with the switch in Privacy, for Play, iPhone and website builds (it rides on the update check already disclosed); off until ticked in the F-Droid build. Or opt-in everywhere?
2. **F-Droid and the update check.** F-Droid names update checks without permission as *Tracking*. Should the F-Droid build ask before its first update check, or rely on F-Droid's own update handling and drop the check there?
3. **Google's request logs.** Exclude Cloud Run request logs for `/v1/app-status` and `/v1/crash-reports` with a Cloud Logging exclusion? The policy could then say no address is logged anywhere for them, at the cost of security logs for those two routes.
4. **Threshold.** Hide counts under 10 (proposed) or under 5?
5. **Daily figures by country.** Proposed: never (month only).
6. **Symbols.** Store release symbol files in R2 (proposed) or elsewhere, and run the Dart decode job on Cloud Run?
7. **Native crashes.** Leave out, or add a native handler later (with its own F-Droid check)?
8. **Play Console and F-Droid numbers.** Pulling Play install figures needs Google credentials on the server; proposed: no, read them in Play Console.
9. **Notices from the dashboard.** Add a form to send and stop notices (passkey-protected), or keep the admin token API?
10. **Access.** Only the owner's passkeys (proposed), or also a named maintainer?
11. **Federated learning.** Should the aggregator store rounds, cohort sizes and privacy spend so the dashboard can show them?
