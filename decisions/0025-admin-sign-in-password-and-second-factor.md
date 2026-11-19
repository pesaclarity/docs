# 0025. Admin sign-in: a password and a second factor

Date: 2026-11-19 · Status: Accepted · Amends [0024](0024-counting-installs-and-crash-reports.md) §4 (sign-in) · Builds on [0004](0004-eu-hosting-and-email-sign-in.md)

This is a plan. Effort figures are estimates, not measurements. The owner accepted it on 2026-11-19, approved the mockups and took every recommendation; the answers are in [Owner's decisions](#owners-decisions-2026-11-19). 0024 §4 carries an amendment pointing here.

## Context

The owner asked for "a password and username and a layer for MFA with Brevo and authenticator app and rate limiting", and said passkey-only sign-in "is not sufficient". The owner then set the first sign-in:

1. A starter admin user, made without a hard-coded or well-known password, signs in first.
2. Straight after that sign-in, the session can do nothing until the password is changed.
3. Then the session must add an authenticator app (TOTP, by QR) and confirm a code, sees recovery codes once, and only then opens the dashboard.

What exists today (`pesaclarity-backend`, `origin/dev`, and the redesign in PR #61):

- **Sign-in** (`ngome.admin.passkeys`): discoverable passkeys, user verification required, against an allow-list (`admin_passkeys`). The owner has two. A sign-in sets a random id in a `__Host-ngome_admin` cookie (`Secure`, `HttpOnly`, `SameSite=Strict`) for 12 hours (`NGOME_ADMIN_SESSION_HOURS`, at most 24); the server keeps only its SHA-256. Every POST checks `Origin` against `NGOME_ADMIN_ORIGIN`.
- **Enrolment**: `POST /v1/admin/passkey-enrolments` with `NGOME_ADMIN_TOKEN` mints a single-use link valid for 10 minutes. So today **the admin token alone opens the dashboard**: whoever holds it can enrol a passkey.
- **Exports**: PR #61 adds `/admin/installs.csv`, `/admin/adoption.csv` and `/admin/monthly.csv`. There are no settings pages; notices are still sent with the admin token.
- **Email codes for the app** (`ngome.auth.service`): `generate_otp` (6 digits from `secrets`), `hash_otp` (HMAC-SHA-256 under `NGOME_CONTACT_PEPPER`, bound to the challenge id), `constant_time_equals`, 5-minute expiry (`otp_ttl_seconds`), 5 attempts per code (`otp_max_attempts`), 5 codes per address per hour (`otp_max_per_hour`). Mail goes through `SmtpEmailSender` to Brevo (`NGOME_SMTP_*`, `NGOME_EMAIL_FROM`).
- **Rate limiting today**: the per-address and per-code limits above; at most 200 open passkey ceremonies at once (`MAX_OPEN_CHALLENGES`, global); the counting flood ceiling (`FloodGate`, kept in memory per instance, by design). Nothing limits by client address, and nothing is shared across Cloud Run instances except what sits in Postgres.
- **Password hashing**: none. Argon2 is not a dependency.
- **The client address.** Cloud Run sets `X-Forwarded-For`, and `FORWARDED_ALLOW_IPS` is `*`. With `*`, Uvicorn takes the *leftmost* entry, which the caller can write. Any per-address limit built on `request.client.host` could be dodged by sending a made-up header. (The same applies to the country lookup in counting; that is a small, separate fix.)
- **Running**: Cloud Run, 512 MiB, at most 3 instances, Neon Postgres; settings in GitHub's `production` environment, synced by the **Sync settings** workflow.

## Decision

Sign-in becomes **username and password, then a second factor, every time**. The authenticator app (TOTP) is required and is the usual second factor. Passkeys stay as an optional, phishing-proof second factor. An emailed code (Brevo) is a fallback, and recovery codes are the last resort. A passkey never replaces the password. The first sign-in uses a starter user made with the admin token, then forces a password change and the authenticator set-up. Limits, lockouts and the audit log live in Postgres. Sessions get shorter, and a fresh second factor is needed before CSV downloads and before any change to sign-in settings.

### 1. The starter user and the first sign-in

**Making it.** With the admin token, from a terminal, the same way passkeys are enrolled today:

```sh
curl -X POST "$API/v1/admin/starter-user" -H "Authorization: Bearer $NGOME_ADMIN_TOKEN" \
  -H "Content-Type: application/json" -d '{"username": "owner"}'
# 201 {"username": "owner", "temporary_password": "<24 random characters>",
#      "expires_at": "<24 hours from now>", "sign_in_url": "https://…/admin/sign-in"}
```

- The temporary password is 24 characters from `secrets` (about 140 bits), printed once in the response, never emailed, never logged, never in code, the image, the repo or an environment variable. The server stores only its Argon2id hash, marked temporary, with an expiry 24 hours ahead (`NGOME_ADMIN_STARTER_HOURS`).
- The username is chosen by the owner on the command: 3 to 32 characters, `a-z`, `0-9`, `.`, `_`, `-`, stored lower-case. Not the email address, and `admin`, `root`, `administrator` and `pesaclarity` are refused, so nothing about it is a well-known default.
- **Run again** while only a temporary password exists (unused, expired or half-way through the flow): the old one stops working and a new one is printed. **Run again once a real password exists**: refused with `409` unless the body says `"reset": true`. A reset replaces the password with a new temporary one, deletes the authenticator and the recovery codes, ends every session, keeps the passkeys, and emails the owner at once. One admin user exists at most.
- The route keeps the admin token's existing check (constant time, `404` when unset) and gains a failure limit by client address (§4).

**The first sign-in** (mockups 01 to 05):

1. **Sign in** with the username and the temporary password.
2. **Email check** (recommended, decision 1): a 6-digit code goes by Brevo to `NGOME_ADMIN_EMAIL`. This is the one place the design adds to the owner's flow, and it is why a leaked admin token alone no longer opens the dashboard (§6). The temporary password is used up once this step passes.
3. **Choose your password** (§2). The temporary password stops working.
4. **Set up your authenticator**: a QR code drawn as inline SVG on the server (`segno`, BSD-3-Clause; inline SVG passes the existing `Content-Security-Policy`, where a `data:` image would not), the key written out for typing, and a code from the app to confirm. Nothing is saved until a code verifies.
5. **Recovery codes**, shown once, with download and copy; **Open the dashboard** unlocks once the owner ticks that they saved them.

**Enforced on the server.** A session from a starter sign-in carries a *stage*: `email`, `password`, `authenticator`, `recovery_codes`, then `full`. The `SignedIn` dependency admits only `full`; every other stage gets `303` to the page for the step still owed, for every page, every CSV and every POST (a JSON `403` naming the step for script calls). A staged session lasts 30 minutes at most, and its id is replaced at each stage change. If the owner walks away part-way, the temporary password is already used up, so they run the command again. Every limit in §4 applies to this flow too.

### 2. First factor: username and password

- **Hashing**: Argon2id with `argon2-cffi` (MIT), at OWASP's first listed setting, m = 46 MiB, t = 1, p = 1, with a 16-byte salt and a 32-byte tag. A per-instance semaphore lets at most 2 hashes run at once, so a burst cannot exhaust 512 MiB. `check_needs_rehash` upgrades stored hashes when the setting changes. An unknown username is checked against a fixed dummy hash, so timing doesn't reveal whether the name exists.
- **Rules** (NIST SP 800-63B): at least 15 characters, at most 128; any characters, normalised to NFKC; no rules about symbols or digits; no expiry on a timer. Refused: a password containing the username, `pesaclarity` or the dashboard's host name, and any password on a bundled list of the 100,000 most common passwords.
- **Breached-password check, offline.** A full check against the Have I Been Pwned corpus offline is **not possible** here: the full corpus is tens of gigabytes, and even a Bloom filter of it at a 1-in-1,000 false-positive rate is over a gigabyte, beyond the image and the 512 MiB instance. The k-anonymity range API would be an external call, which this design rules out. So the check is the common-password list (about 1 MB, loaded into a `frozenset`); candidates are the NCSC *100k most-used passwords* list as carried by SecLists (MIT), its licence to be confirmed before use (decision 8). The 15-character minimum and a required second factor carry the rest; the page recommends a passphrase from a password manager.
- **Changing it**: in Security, with the current password and a fresh second factor (not an email code). Every other session ends; the owner is emailed.
- **Forgotten**: the starter command with `"reset": true` (§1). No email reset link exists, ever.

### 3. Second factors

| Factor | Role | Notes |
| --- | --- | --- |
| Authenticator app (TOTP, RFC 6238) | **Required**, set up at first sign-in; the usual second step | SHA-1, 6 digits, 30-second steps (what every app supports); accepts the current step and one either side. **Replay**: the last accepted step is stored per user, and a code for that step or an earlier one is refused, so each code works once. The 160-bit secret is stored encrypted with AES-256-GCM under `NGOME_ADMIN_SECRET_KEY` (the `cryptography` package, already installed through `pyjwt[crypto]`), with the user id as associated data. RFC 6238 is about 40 lines; written in-house and tested against the RFC's vectors, so no new dependency. **Replace** shows a new QR and keeps the old secret until a code from the new one verifies. |
| Passkey | **Optional, recommended**; the only phishing-proof factor | The owner's two passkeys stay. After the password, the page asks for one of the user's credentials (`allowCredentials`), user verification still required. Added from Security after a fresh second factor; the token-only enrolment route is removed. |
| Email code (Brevo) | **Fallback** | 6 digits to `NGOME_ADMIN_EMAIL`, made, hashed and checked with the app's own primitives (`generate_otp`, `hash_otp`, `constant_time_equals`) in a separate `admin_email_codes` table bound to the sign-in in progress, so an admin code can never be used on `/v1/auth/email/verify`. Expires in 5 minutes, 5 tries per code. Sent only after a correct password. The email holds the code and the dashboard's host name, never a link; the wording is fixed in code. A session signed in this way can read and download, but cannot change sign-in settings (decision 3). |
| Recovery codes | Last resort | 10 codes of 10 characters (Crockford base32, about 50 bits each, shown as `XXXXX-XXXXX`), stored as HMAC-SHA-256 under the pepper, each usable once, shown once. **Make new codes** replaces all ten. Using one emails the owner; Security shows how many are left. |

**Order on the page.** After the password, the authenticator code page opens directly (mockup 07). **Use another way** lists a passkey (marked strongest), then email, with a recovery code below.

**Optional, not proposed now:** "on a browser never seen before, accept only a passkey or a recovery code". That stops real-time phishing of the password and a TOTP or email code, but it would make the passkey the everyday factor on any new laptop, which is not what the owner asked. Revisit if the dashboard ever holds more than counts.

### 4. Rate limits, lockouts, audit and alerts

**The client address.** A helper reads `X-Forwarded-For` from the right, skipping `NGOME_ADMIN_PROXY_HOPS` entries (default 1: Cloud Run's front end appends the caller's address last). Before relying on it, a request with a made-up header goes to the live service to confirm which entry Google writes. The address is never stored: limits and the audit log use an HMAC of it under the pepper (`admin-ip` domain), plus a country from the DB-IP database already in the image.

**Limits** live in one Postgres table, `admin_throttle (bucket, failures, window_start, locked_until, lockouts, updated_at)`. `bucket` is an HMAC of the scope and value. Each check and update is one `INSERT … ON CONFLICT DO UPDATE … RETURNING` inside the request's transaction, so all instances share the count and a race cannot slip past it. Defaults, each a setting:

| What | Bucket | Limit | Then |
| --- | --- | --- | --- |
| Wrong password | username, from browsers not seen before | 5 failures | Paused 15 minutes; each later pause within 24 hours doubles, up to 24 hours |
| Wrong password | username, from a known browser (device cookie, §5) | 10 failures | Paused 15 minutes, doubling. Someone else's guessing cannot lock the owner out of a browser they already use |
| Wrong password | client address | 20 failures an hour | Paused 1 hour, doubling |
| Wrong second-factor code (TOTP, email, recovery) | the sign-in in progress | 5 | That sign-in ends; start again from the password |
| Wrong second-factor code | user | 10 in 24 hours | Second step paused 1 hour, doubling; the owner is emailed at the 3rd failure |
| Email code sends | the sign-in in progress | 1 a minute, 3 in all | **Send a new code** waits |
| Email code sends | user | 5 an hour, 10 a day (`otp_max_per_hour` reused) | No more codes until the window passes |
| Admin token failures | client address | 20 an hour | `429` for an hour |
| Every password failure | global | 200 an hour | No lockout (that would be a denial of service); one alert email |

Buckets exist for usernames that don't, so answers never reveal whether a name is real. Messages are generic: "That username and password don't match" for any first-step failure; "That code didn't work" for any second-step failure; and on a pause, "Sign-in is paused" with the time it ends (mockup 11). A pause doesn't say whether the username exists. The purge job deletes buckets idle for 48 hours.

**Audit log** (`admin_events`): time, kind (password ok or failed, second factor ok or failed and which, email code sent, paused, starter user made or reset, password changed, factor added, replaced or removed, recovery code used, codes remade, session ended, CSV downloaded, re-authenticated), outcome, address HMAC, country, browser and system family ("Firefox on Linux"), and whether the browser was known. Never a password, code, secret, submitted username, full address or cookie. Kept 90 days (the app's `retention_auth_event_days`). Security shows the latest entries.

**Email alerts** to `NGOME_ADMIN_EMAIL`, at most one of each kind an hour: sign-in paused; right password, then wrong second factors; sign-in from a browser not seen before; password, factor or recovery-code changes; a recovery code used; the starter command run (made or reset). Alerts go to the same mailbox as fallback codes, so a thief who holds the mailbox can delete them; the audit log keeps the record.

### 5. Sessions

- **Length**: 8 hours from sign-in (down from 12), and 30 minutes idle (`NGOME_ADMIN_SESSION_HOURS`, new `NGOME_ADMIN_IDLE_MINUTES`). Last-seen is written at most once a minute.
- **Re-authentication**: a fresh second factor (no password) opens a 10-minute window (`NGOME_ADMIN_REAUTH_MINUTES`) for CSV downloads (mockup 13), and for sign-in changes: password, factors, recovery codes and signing out other sessions. For CSV downloads an email code is accepted; for sign-in changes it is not.
- **Cookies**, all `__Host-`, `Secure`, `HttpOnly`, `SameSite=Strict`, `Path=/`: `ngome_admin` (the session, id replaced at sign-in, at each first-sign-in stage and at re-authentication); `ngome_admin_pending` (the 5 minutes between password and second factor, server-side state only); `ngome_admin_device` (a random id kept 400 days, stored as a digest, used only for "browser not seen before" and the known-browser bucket). `Origin` checks on every POST stay.
- **Security page** (mockup 12): username and password; recovery codes left; second factors; signed-in browsers with **Sign out** and **Sign out everywhere else**; alerts; recent activity. A **Security** item joins the sidebar and the phone's tab bar from PR #61.

### 6. Threat model

| Threat | Today (passkey only) | With this design | What is left |
| --- | --- | --- | --- |
| Phishing (a look-alike page relaying in real time) | Passkeys are bound to the origin; phishing fails | A relayed password plus a TOTP or email code would work once; a passkey still can't be relayed. The emails name the real host and never carry a link; sessions are short and every new browser alerts | Real. The way out is a passkey, which stays available; see "not proposed now" in §3 |
| Credential stuffing or guessing | No password to stuff | A unique password of 15 or more characters, the common-password list, Argon2id, limits by username and address, generic errors, and a second factor behind it all | Low |
| Stolen email account | No effect | The thief also needs the password; email-code sessions can't change sign-in settings; alerts and the audit log record the attempt | They can read the counts if they also have the password |
| Stolen or lost phone | Its passkey is behind the phone's screen lock | The thief also needs the password. The owner signs in elsewhere with a recovery code or the other passkey, replaces the authenticator, removes that passkey and signs out everywhere else | An unlocked phone that also holds the password in a password manager |
| Leaked admin token | **Opens the dashboard** (enrol a passkey) | The token can make or reset the starter user, but the first sign-in also needs the email code (decision 1), and the owner is emailed at once. The token-only passkey route is removed | The token still sends app notices, as today; rotate it with `gh secret set NGOME_ADMIN_TOKEN` and **Sync settings** |
| Database copy | Digests only | Argon2id hashes, HMACs, an AES-GCM-encrypted TOTP secret whose key is not in the database | An offline guess at a 15-character random password is not practical |

### 7. Data, settings and dependencies

New tables: `admin_users` (username, Argon2id hash, temporary flag and expiry, password changed at), `admin_totp` (ciphertext, nonce, last accepted step), `admin_recovery_codes`, `admin_email_codes`, `admin_pending` (the sign-in between steps), `admin_devices`, `admin_throttle`, `admin_events`. `admin_sessions` gains `stage`, `factor`, `device`, `last_seen_at` and `reauth_until`, and points at the user rather than a passkey.

New settings:

| Setting | Kind | Value |
| --- | --- | --- |
| `NGOME_ADMIN_EMAIL` | variable | The owner's address for codes and alerts. Required once the dashboard is on |
| `NGOME_ADMIN_SECRET_KEY` | **secret** | Base64 of 32 random bytes, for the TOTP secrets: `python3 -c 'import secrets,base64; print(base64.b64encode(secrets.token_bytes(32)).decode(), end="")' \| gh secret set NGOME_ADMIN_SECRET_KEY --env production`, and named in `sync-settings.yml`. Losing it means setting up the authenticator again (reset) |
| `NGOME_ADMIN_PROXY_HOPS` | variable, optional | Default 1; change only if the forged-header test says so |
| `NGOME_ADMIN_SESSION_HOURS`, `_IDLE_MINUTES`, `_REAUTH_MINUTES`, `_STARTER_HOURS`, and the limits in §4 | variables, optional | Defaults as written |

Brevo needs no change: the same SMTP login and sender. New dependencies: `argon2-cffi` (MIT) and `segno` (BSD-3-Clause), plus the common-password list file. The README's "what we store" row for the dashboard changes (password hash, encrypted TOTP secret, hashed codes, hashed addresses, audit entries). The app's privacy policy is unaffected: none of this is about people who use the app.

### 8. Migration

1. The owner sets `NGOME_ADMIN_EMAIL` and `NGOME_ADMIN_SECRET_KEY` and runs **Sync settings**.
2. The release's migration creates the tables and deletes every existing admin session. Passkey-only sign-in ends with that release; until a user exists, `/admin/sign-in` says to run the starter command.
3. The owner runs the starter command and goes through the first sign-in (§1).
4. The two enrolled passkeys stay in `admin_passkeys` with their labels and counters, and show in Security as second factors. Nothing has to be enrolled again.
5. `POST /v1/admin/passkey-enrolments` is removed; `GET` and `DELETE /v1/admin/passkeys` stay (they can only list or remove).

### 9. Mockups

Static HTML and CSS on PR #61's look (sidebar shell, pure palette, Inter and Space Grotesk served locally), light and dark, at 390 and 1280 pixels: first sign-in (01), email check (02), forced password change (03), forced authenticator set-up with QR, key and code (04), recovery codes (05), later sign-in (06, and 06b with the generic error), authenticator code (07), another way (08), email code (09), recovery code (10), paused (11), Security (12), and confirm it's you (13). The data is synthetic; the QR code holds an example key.

### 10. Stages

| Stage | Work | Effort (estimate) |
| --- | --- | --- |
| 0 | Owner answers the decisions below and approves the mockups; confirm the password list's licence; the forged-header test | owner time |
| 1 | Tables and migration, settings, client address helper, `admin_throttle` and its tests (including two sessions racing) | 2 days |
| 2 | Starter command, Argon2id, password rules and list, staged sessions and the redirects | 2 days |
| 3 | TOTP with QR and replay protection, recovery codes, admin email codes, passkey as a second factor | 2.5 days |
| 4 | Idle timeout, re-authentication, device cookie, audit log, alert emails, purge | 1.5 days |
| 5 | Templates for the 13 screens, Security page, the small script for passkeys, copy and download | 2 days |
| 6 | End-to-end tests, README, `deploying.md`, the sync workflow, roll-out | 1.5 days |

About **11 to 12 days**. It ships as one release: passkey-only sign-in must not end before the new flow exists.

## Consequences

- The dashboard needs something the owner knows and something the owner has, every time, as asked. The admin token alone no longer opens it.
- The usual sign-in (password and TOTP) can be phished in real time, which passkey-only could not. Passkeys stay available, so the owner can use the phishing-proof path when they choose.
- The owner keeps two more secrets: a password and the recovery codes. The server keeps a new secret key and an email address.
- Admin sign-in now stores hashed client addresses and coarse browser details for 90 days.
- More code to own: about 1,500 lines with tests, and two small dependencies.
- The counting module's country lookup trusts the leftmost `X-Forwarded-For` entry; it should use the same helper (a separate small fix).

## Owner's decisions (2026-11-19)

The owner accepted this ADR, approved the mockups and took each recommendation:

1. **Email check at the first sign-in.** Yes: the starter sign-in also needs a code emailed to `NGOME_ADMIN_EMAIL`.
2. **Passkeys.** Kept as an optional, phishing-proof second factor, always after the password.
3. **Email code.** An everyday fallback that can read and download but not change sign-in settings.
4. **Session length.** 8 hours, ending after 30 minutes idle.
5. **Fresh second factor before CSV downloads.** Yes, for a 10-minute window, as for sign-in changes.
6. **Limits.** As in §4.
7. **Alerts.** All six kinds in §4.
8. **Password rules.** At least 15 characters and a bundled list of common passwords, its licence confirmed before use.
9. **Reset scope.** `"reset": true` replaces the password and wipes the authenticator, the recovery codes and every session, keeps the passkeys, and still needs the email check.
10. **Audit retention.** 90 days, with addresses only as keyed hashes and a country.
