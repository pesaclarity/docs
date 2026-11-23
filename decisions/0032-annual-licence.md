# 0032. Premium features with an annual licence

Date: 2026-11-23 · Status: Proposed · Builds on [0004](0004-eu-hosting-and-email-sign-in.md) and [0031](0031-group-budgets.md)

Agreed with the owner in outline. Nothing here is built yet. Prices are not set.

## Context

Pesa Clarity needs income without ads, selling data or sending transactions to our server. The app works offline, so a paid plan must also work offline.

## Decision

### Free and premium

- **Free:** personal finance visibility.
- **Premium candidates:**
  - shared spaces with approvals (chamas, households)
  - business tools: stock, invoices, tax pack, reports
  - many lines and currencies

| Plan | Price per year |
| --- | --- |
| Premium | TBD |
| Business | TBD |

### The licence

- After payment, our server issues a small licence signed with Ed25519. It holds the account id, plan, features and expiry date.
- The app checks the signature offline with a public key built into the app.
- A licence lasts one year. Renewal issues a new one.

### Checks

- **Daily, offline:** the app checks the expiry date.
- **Quarterly, online:** the app asks our existing backend whether the licence was refunded or revoked. The reply includes a timestamp signed by the server.

### Clock tampering

- The app keeps a "latest time seen" in the encrypted database. It never goes backwards.
- SMS message dates after the expiry date prove the licence has expired.
- A signed server time moves "latest time seen" forward.
- If the phone's clock looks wrong (earlier than "latest time seen"):
  - a 7-day grace period starts at once, with the message: "Your phone's date looks wrong. Fix it or go online to keep premium features."
  - after 7 days without a good check, premium features become read-only. Data is never locked.
  - one successful online check unlocks them.
- If the phone is offline at the quarterly check and the clock looks fine: 30-day grace period.

### Repeated rollbacks

- 3 clock rollbacks within 90 days: the server flags the licence. The next check locks premium features until the user emails support. Support can unlock in one tap.
- No automatic bans. Manual bans only for clear abuse: one licence shared across many phones, or chargeback fraud.

### Expiry

Premium features become read-only. Export is always available.

### Payment

- **Website:** M-Pesa through CitaPay (or Paystack).
- **Play build:** Google Play billing, which Google requires for digital features.
- Both produce the same licence.

### Open source and F-Droid

- F-Droid users buy the same licence on the website.
- Anyone building the open code can remove the check. We accept that.
- To check: what this means for F-Droid anti-features.

### Shared spaces

Only the space owner needs a licence. Members join free.

### Later: server sync

Group sync (0031) moves to server sync as a paid feature, with the same end-to-end encryption: the server stores only ciphertext, using the same keys and signed approvals as 0031 stage 1. Server costs stay low.

## Accepted weaknesses

- A phone that stays offline and blocks SMS reading can keep premium features past expiry.
- A patched build can skip the check.

Both are acceptable.

## Consequences

- Premium works offline; our server never sees transactions.
- New server work: licence signing, the quarterly check and support unlock.
- Two payment routes to maintain.
