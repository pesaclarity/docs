# 0033. Group sync server on celld

Date: 2026-11-25 · Status: Accepted; hosting replaced by [0035](0035-group-sync-through-the-backend.md) · Builds on [0031](0031-group-budgets.md) and [0032](0032-annual-licence.md)

Not agreed with the owner yet. Nothing here is built.

## Context

0031 stage 1 shares a space through encrypted `.pcshare` files (X25519 key exchange, AES-256-GCM, a change log ordered by hybrid logical clocks, signed approvals). It works, but members must pass files around by hand. 0032 makes server sync a paid feature for the space owner. We need a server that is cheap when idle, simple to run and cannot read the data.

## Decision

Run our own [celld](https://github.com/denoland/celld) (v0.6.x). celld is a self-hosted runtime for Cloudflare-style Workers and Durable Objects. Each "cell" is a Durable Object with its own SQLite database. State lives in an S3-compatible bucket. Each cell has a single owner node at a time, with failover to another node. Idle cells cost almost nothing.

### One cell per shared space

- Each shared space gets one cell, named by the space id.
- The cell keeps the space's change batches, members' public keys and a cursor per batch.
- The server stores and relays **ciphertext only**. Batches use the same encryption as `.pcshare` files. The server has no keys and cannot read transactions, names or amounts.

### Who can talk to a cell

- Members sign each request with their space member key (Ed25519, as in 0031). The cell checks the signature against its member list.
- The member list changes only through a signed approval from the space owner, as in 0031.
- The space owner's signed licence (0032) is sent when the space is created and on renewal. The cell checks it offline with our public key. If it has expired, the cell goes read-only: members can still pull, nobody can push.

### Sync

- **Push:** a member sends a batch of encrypted changes. The cell gives it the next cursor.
- **Pull:** a member asks for everything since its last cursor.
- **Live updates:** an open app keeps a WebSocket. The cell uses WebSocket hibernation, so idle connections cost nothing. Phones that are closed just pull next time.
- Merging still happens on the phone, by HLC order, exactly as in 0031. The server does not merge.

### Retention and compaction

- Batches are kept until every member has pulled past them, and for at least 90 days.
- A member can upload an encrypted snapshot. Batches older than the newest snapshot that everyone has seen are then deleted.
- New members start from the latest snapshot.

### Deleting a space

The owner deletes the space: the cell is deleted along with all its bucket objects. Nothing is kept on our side.

### What the server sees

The server sees the space id, member key ids, batch sizes and times, and IP addresses. To keep this small:

- space ids and key ids are random, not linked to email or account id;
- the licence is checked in the cell, and only "valid until" is stored;
- batches are padded to fixed size steps;
- logs keep no request bodies and no IPs beyond 7 days.

## Operations

- celld is **beta**. We put TLS in front of it (Caddy, or the Cloud Run load balancer) and keep peer traffic between celld nodes on a private network.
- **Bucket:** Google Cloud Storage, because our backend already runs on Cloud Run. Cloudflare R2 is the fallback (no egress fees).
- **Costs:** two small nodes plus bucket storage. Expect a few dollars a month at launch; storage grows with snapshots, not with members.
- **Backups:** bucket versioning plus a daily copy to a second bucket. Restoring a cell restores ciphertext only.
- **Monitoring:** node health, cell count, push/pull errors and latency, bucket size. Alerts to the same place as the backend.

## Alternatives considered

1. **Cloudflare-hosted Durable Objects.** Same model, no ops. Rejected for now: cost per request at scale and lock-in to one provider. celld runs the same code, so we can move later.
2. **Our Cloud Run backend with Postgres.** Familiar, but we would build relay, live updates and per-space isolation ourselves, and run a database. More ops.
3. **Keep file-only sync.** No server cost, no new risk. It stays as the free tier; it does not give live sync.

## Risks

- **celld maturity.** It is v0.6 and beta. Bugs or breaking changes are likely. Mitigation: pin versions, test upgrades on staging, keep `.pcshare` files working as a fallback.
- **Single vendor project.** If celld stops, we must move. Mitigation: the code targets the Workers/Durable Objects API, which runs on Cloudflare too.
- **Metadata.** Even ciphertext-only sync shows who syncs when. We accept the list above.

## Stages

1. Prototype: one celld node, local bucket, push/pull, signed requests. Tests against 0031 merge rules.
2. Staging: two nodes, GCS, TLS in front, licence check, snapshots, monitoring.
3. Beta for paid owners who opt in, with `.pcshare` still available.
4. General release; WebSocket live updates on by default.

## Consequences

- Paid owners get live sync without our server ever seeing transactions.
- New things to run: celld nodes, a bucket, monitoring.
- The app needs a sync client alongside the file import/export from 0031.

## Amendment, 2026-11-28

Group sync is not hosted on celld. The owner decided that it goes through the existing backend: [0035](0035-group-sync-through-the-backend.md) replaces the hosting in this record. There are no celld nodes, no separate domain and no separate bucket, so Operations, the celld risks and the stages above no longer apply, and alternative 2 is what was chosen. The protocol and the privacy rules stay as written here: a server with no keys that never opens a batch, signed requests, the licence check, read-only on expiry, padding, retention, deleting a space and what the server sees. "Ciphertext only" needs one correction, which 0035 spells out: the entries in a batch are sealed, but the batch the app sends today also carries its true length and the file's header in clear. The WebSocket for live updates and the snapshots are not built yet.
