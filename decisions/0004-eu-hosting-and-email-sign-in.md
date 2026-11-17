# 0004. EU hosting and email sign-in

Date: 2026-10-27 · Status: Accepted

## Context

The server holds sign-in details and encrypted backups only, and must stay cheap for an open-source project. Heroku was considered first and dropped on cost. Cloud SQL was ruled out. The owner wants hosting in the EU, not South Africa. SMS sign-in codes cost money per message, which an open-source project cannot carry.

## Decision

Run the API on Google Cloud Run, Postgres on Neon, backup ciphertext on Cloudflare R2 with EU jurisdiction, and send sign-in codes by email through Brevo, all in the EU. Sign-in is by email code, plus Google on the Play build. No SMS codes.

## Consequences

- All four services fit free tiers at proof-of-concept traffic; Cloud Run scales to zero, so the first request after a quiet spell is slow.
- Every name, region and provider setting comes from deployment configuration; none lives in the code.
- People without an email address cannot make an account, but can use the app without one.
