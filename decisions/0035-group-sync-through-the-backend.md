# 0035. Group sync through the backend

Date: 2026-11-28 · Status: Accepted · Replaces the hosting part of [0033](0033-group-sync-server-on-celld.md) · Builds on [0031](0031-group-budgets.md) and [0032](0032-annual-licence.md)

The owner decided this on 28 November 2026. The routes are written and tested in [pesaclarity-backend #77](https://github.com/pesaclarity/pesaclarity-backend/pull/77), which is waiting for review. Nothing is deployed, and nobody can use it yet.

## Context

0033 chose a separate sync server: a Worker on our own celld nodes, behind its own domain, with its own bucket. The Worker was written and tested (pesaclarity-backend, `sync/`). Its hosting was still to be set up.

Hosting it means new things to look after: two nodes of beta software, a domain, a bucket, backups and monitoring. Our backend already runs on Cloud Run, with a Postgres database, a blob store for the vault, a daily purge job and one place for settings. The app already uses the backend's address for sync when no other is set.

0033 turned this option down (its alternative 2) as "more ops". With the backend already running, it is the other way round: celld would be the extra thing to run.

## Decision

**Group sync goes through the existing backend, not through a separate sync server on its own domain.**

- The backend serves the 0033 wire protocol at the same paths (`/v1/spaces/{space}` and below), so the app's sync client needs no change.
- No celld nodes, no separate domain, no separate bucket. `sync/` stays in the repository as the reference and is marked as replaced.

### What stays from 0033

The protocol and every privacy rule:

- The server stores and passes on **ciphertext only**. It has no keys, never parses a batch and merges nothing.
- Members sign each request with their space member key. A forged, stale or repeated request is refused.
- The owner's licence (0032) is checked offline. Only "valid until" is kept. When it runs out the space is **read-only**: pulls work, pushes and new members are refused, and the owner can still remove a member.
- Batches are padded to 1, 2, 4 … 1024 KiB.
- A batch is deleted once every member it is for has pulled past it and it is at least 90 days old.
- When the owner deletes a space, every row and stored batch of it goes.
- Rate limits per member key and per address. Our own logs hold no bodies and no addresses.

### How the backend does it

- **Storage.** Spaces, members, the batch index, cursors and seen nonces are rows in the backend's Postgres. Batches are objects in the vault's blob store, under random keys.
- **Kept apart from accounts.** The sync routes take no sign-in. The sync tables hold no account id or email, and nothing links them to the account tables.
- **Several instances.** Cloud Run runs more than one copy of the server. Each change to a space locks that space's row, so batches get their cursors one at a time; a push can safely be sent twice; one signed request sent to two copies is accepted once.
- **Off until wanted.** Until the licence public key is set, every licence is refused and no space can be made.

## What the server sees

The same as in 0033: the space id, member key ids (who sent each batch and who it is for), batch sizes and times, and the address each request comes from. Ids and keys are random and not linked to an email or an account. The address is used for the flood limit, in memory only.

**New here:** Google's request log for Cloud Run records each request's URL, which holds the space id, and the caller's address, for 30 days by default. 0033 allows no address in logs beyond 7 days. So these routes must be left out of that log before sync is turned on, as 0024 did for the update check and crash reports.

## What is given up

- **Live notices, for now.** 0033's WebSocket told an open app about a new batch at once. It is not built. The app does not use it: it pulls when it opens, when it comes back and on pull-to-refresh. The server answers `/ws` with a clear "not here". It can be added later.
- **One cell per space.** In 0033 each space had its own small database. Here spaces are rows in shared tables, kept apart by the code and its tests, not by the runtime. A wrong query could cross spaces. What it could show is still only ciphertext and the metadata above.
- **A separate machine.** One server now takes sign-in, vault and sync requests. Nothing we store joins a space to an account, but whoever can read request logs could match an address that signs in and syncs at about the same time. Leaving the sync routes out of Google's request log keeps that from being written down.
- **Its own capacity.** A flood of sync requests shares the backend with sign-in and the vault. The limits live in each instance's memory, so with three instances a caller can get three times each.

## Still needed before anyone can use it

1. **Review and merge** of the backend pull request, then **the owner's `dev` → `main` approval and deploy**.
2. **Licence issuing (0032).** Nothing issues a licence yet. The server checks the planned format, and a small command mints test licences.
3. **The app must hold a licence.** A test build with sync on sends the text `dev` today, which the server refuses.
4. **By the owner, on the hosting:** set the licence public key; leave the sync routes out of Google's request log; move the vault's blob store to R2 first (the server reads from one store at a time); put the purge job on the new image.
5. **Privacy policy 2.6** before sync is offered to anyone, as 0031 decided: our server gains a job and holds a new kind of data.

## Consequences

- Nothing new to host. Sync ships, migrates and is rolled back with the backend.
- Sync is up when the backend is up, and the two share its capacity.
- 0033's protocol and privacy rules stand. Its Operations section, stages, costs, bucket and celld risks no longer apply.
- If sync outgrows the backend, the Worker in `sync/` speaks the same protocol, so a later move would not change what the phones send.
