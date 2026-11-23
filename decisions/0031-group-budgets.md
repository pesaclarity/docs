# 0031. Group budgets: one Household space on several phones

Date: 2026-11-23 · Status: Proposed · Builds on [0004](0004-eu-hosting-and-email-sign-in.md), [0016](0016-spaces-kept-apart.md), [0018](0018-double-entry-journal.md), [0027](0027-money-between-spaces.md) and [0029](0029-firefly-iii-sync.md)

This record proposes a direction and asks the decisions at the end. The owner must approve it before anything is built. Nothing here is built. Screens wait on an approved mockup.

## Context

A family, a couple or a chama wants one budget they can all see and add to, each from their own phone. Today Pesa Clarity cannot do that. Every example below is made up.

> Achieng and Otieno share rent, food and school fees. Each has their own M-Pesa line. They want one Household budget: "Food KES 18,000 a month, KES 11,400 spent so far", with both their food payments counted, and neither seeing the other's personal spending.

What exists today (`pesaclarity-app` `origin/dev`, privacy policy 2.5):

- **Everything stays on the phone.** Transactions, messages, statements and Ask stay on the phone. The privacy policy says our server "never sees a transaction". The only exception is Firefly III sync (0029), which goes from the phone to the person's own server, never ours.
- **Our server does narrow jobs** (0004): sign-in by email code or Google, and the vault. It also answers the update check, model updates and crash reports the person chooses to send. No ads, and data is never sold.
- **The vault** holds backups encrypted on the phone (`.pcbackup`: AES-256-GCM, a key from the person's passphrase with Argon2id, and a recovery key). The server keeps the latest 10 per space and cannot read them. Each upload is checked as coming from the genuine app (Play Integrity, or App Attest on iPhone).
- **Spaces are separate encrypted databases** (0016). Each has its own SQLCipher file, opened with one device key. Data is never read or copied from one space into another. Household is already a kind of space, with its own categories.
- **An account lives in one space** (0027). A transaction can be switched to another space and then shows only there. Where the account lives keeps a balance record only (amount, time, fee, code, reported balance), with no name, category or note.
- **The journal** (0018) is the source of truth for money. Transaction ids are numbers unique only inside one space, so two phones would hand out the same ids.
- **Work that waits for a connection** already has a pattern (`BackupWhenOnline`), and one-way sending of a space already has a design (0029).

The question: how can several phones share one Household space, or at least one budget, without our server ever seeing a transaction, and without breaking what 0016, 0018 and 0027 decided?

## Options

### 1. A shared Household space, end-to-end encrypted, through our server

The Household space lives on every member's phone, as its own SQLCipher file, like any space. Members' phones exchange changes through our server, encrypted on the phone the same way as the vault. The server stores and forwards sealed changes and never sees plaintext.

**Members, invites and roles.**

- A shared space has a **space key**, made on the phone of the person who creates it. Changes are sealed with it (AES-256-GCM, as the vault does).
- To invite someone, the owner of the space shows a QR code or a link with a one-time code. It carries an invite id and a secret, never the space key itself. The new member's phone makes a key pair; the owner's phone checks the secret and seals the space key to that member's public key. The server passes the sealed key on and cannot open it. Both phones show a short safety number (for example "4 7 2 9 1 8") so the two people can check, face to face or on a call, that nobody is in the middle.
- Every member signs in (0004). The server knows only which accounts belong to which shared space id, so it can pass changes to the right phones. It knows no names, no space name and no kind.
- Roles: **Owner** (invites and removes, changes the budget, can close the space), **Member** (adds and edits transactions and budget lines), and **Viewer** (sees, adds nothing). A chama might want a **Treasurer** later; the record keeps three roles to start.
- Removing a member makes a new space key on the owner's phone and seals it to the members who stay. The removed phone keeps what it already had (nothing can take back data already on a phone) but gets no new changes. The app says this plainly before removing someone.

**What each member's own SMS contributes.** Only what they choose to share into the space. This is the same switch 0027 already defines: on a payment read on their own M-Pesa, a member picks "Share to Household". The Household then gets the transaction (amount, time, payee, category, note) on that member's owner's account ("Otieno's M-Pesa"), and their own space keeps a balance record only. Nothing else from their phone goes anywhere: not their other transactions, not their balance, not their messages, not their other spaces. A member can also type a cash payment straight into the Household. A rule like "always share payments to Naivas to Household" could come later, but every share starts as the person's own choice.

**Merge and conflicts.** Each phone keeps a log of changes (operations), not a copy of the whole database.

- Each record in a shared space gets a random id (a UUID) instead of the space's running number, so two phones never clash on ids.
- Each change is one operation: add a transaction, change its category, change a budget line, remove a transaction. Each carries the member's device id and a counter (a Lamport clock), so every phone puts operations in the same order.
- Adding transactions never conflicts: two people adding at once both land.
- Two people changing the same field of the same record: the later one by that order wins, and the app shows "Achieng changed Food from KES 18,000 to KES 20,000" in the space's history, so nobody's change disappears unseen.
- A removal wins over an edit made at the same time, and shows in the history with a way to bring it back.
- The journal (0018) is rebuilt on each phone from the transactions, so balances and totals are never merged, only worked out. That keeps the double-entry rule intact.
- A full CRDT library (Automerge, Yjs) is not needed: the records are rows with a few fields, and last-writer-wins per field with an add-wins set for transactions covers them. If text fields such as shared notes are wanted later, that can change.

**Offline.** Fully works offline. Changes wait on the phone and go when it is online, through the same pattern as `BackupWhenOnline`. Two phones offline for a week merge when they next connect.

**Privacy.** Our server sees, per shared space: a random space id, which signed-in accounts are in it, sealed change blobs with their size and time, and members' public keys. It never sees an amount, a name, a category or the space's name. Members do see each other's shared transactions in full, which is the point.

**Effort.** Large. New: sync endpoints and storage on the server, key handling and invites on the phone, UUID ids and an operation log for shared spaces, a merge engine, member and role screens, history, and the 0027 switch extended across phones. Months, not weeks.

### 2. Share only a budget summary

Each member's phone works out, once a week, totals per category for their own shared categories ("Food KES 4,200, Transport KES 1,150"), seals them with a space key and sends them through the server. Each phone shows the household's totals next to the budget.

- **Privacy.** Less leaves the phone than option 1: no payees, no times, no notes. The server sees the same as in option 1 (sealed blobs, member list). But totals still say a lot, and a household of two can work out each other's totals.
- **Effort.** Medium. Invites and keys are the same work as option 1; the merge is trivial (each member owns their own total).
- **Offline.** Works; totals are a week late at worst.
- **Conflicts.** None: each member writes only their own totals.
- **Limits.** It is not one budget. Nobody can see which payment made Food go over, fix a wrong category, or add a cash payment for the house. It answers "how are we doing" but not "what happened".

### 3. Share by exporting and importing a file

A member exports a Household file (an encrypted `.pcbackup`-style file of shared transactions, with a passphrase) and sends it by WhatsApp, email or Bluetooth. Another member imports it, and the app merges by UUID.

- **Privacy.** Our server is not involved at all. But the file goes through whichever app people use to send it, and the passphrase has to be shared some other way.
- **Effort.** Small to medium: export and import exist; adding UUIDs and a merge on import is the new part, and it is the same merge as option 1.
- **Offline.** Fully, by design.
- **Conflicts.** The same rules as option 1, applied at import. But people forget to send, and import the same file twice, or an old one. Each phone is always behind.
- **Limits.** It works for a couple once a month; it does not work for a chama of 15.

### 4. Don't build it

Members keep their own budgets, and agree on a shared one outside the app.

- **Privacy, offline, conflicts.** Nothing changes.
- **Effort.** None.
- **Cost.** Families, couples and chamas are a large part of who uses money together in Kenya, and the Household and Savings group kinds already exist. Without sharing, those kinds stay one person's view of a shared thing.

## Recommendation

**Option 1, built in stages, starting with a Household space shared by up to a small number of members, with the merge and the 0027 share switch from the start.** Option 3's file is a useful first stage, because it needs the same merge and the same ids with no server work.

Why:

- It is the only option that gives one real budget that everyone can see and fix.
- It keeps the promise that our server never sees a transaction. The vault already proves the approach: encrypted on the phone, server holds only what it cannot read. Sync is the vault with smaller pieces, more than one reader, and a key that is passed between phones instead of typed.
- It fits what 0016 and 0027 already decided. The shared space is still a separate encrypted database; nothing is read across spaces; what a member contributes is exactly what they switch in, and their own space keeps only a balance record.
- It fits the journal (0018), because only transactions are merged and balances are always rebuilt.
- Option 2 needs almost the same key work for much less use. Option 3 alone does not scale beyond two people. Option 4 leaves the Household kind half done.

What it does **not** include, to keep it inside the current architecture: no web view, no reading of other members' messages, no shared accounts (an account still lives in one phone's one space), no server-side search or totals, and no payments or money movement between members.

## Data and privacy impact

- **What leaves the phone:** only transactions a member chose to share into the shared space, and the space's budget lines and categories, always sealed with the space key. Also each member's public key, and sealed copies of the space key.
- **What the server sees:** a random id per shared space, which signed-in accounts are members, their roles (needed to refuse a Viewer's writes; this could also be enforced only on the phones, see the questions), the size and time of each sealed change, and the public keys. Not amounts, names, categories, notes or the space's name.
- **What the server keeps:** sealed changes until every member's phone has them, plus a sealed snapshot so a new member does not need the whole history; then old changes are deleted. A limit applies (for example 90 days for a phone that never comes back).
- **Other members** see every shared transaction in full, including the payee. The app must say so on the share switch: "Otieno and Wanjiru will see this payment."
- **The privacy policy needs a new version (2.6).** "Our server does five narrow jobs and never sees a transaction" stays true in substance, but a sixth job (passing sealed shared-space changes) is new, the server stores a new kind of data (shared-space membership), and other people now see data a person chose to share. The short version, "What our server holds", "Your choices" (leaving a space, what removal can and cannot take back) and deleting an account all change. Stage 1 (file only) needs a smaller update, because nothing new reaches our server, but the policy should still say that a shared file shows its contents to whoever has it and the passphrase.
- No ads, nothing sold: unchanged.

## Effort

Rough stages, each shippable on its own:

1. **Shared ids and the operation log.** UUIDs and an operation log for a Household space, the merge rules, and history. Export and import of a sealed Household file (option 3). No server work. Weeks.
2. **Share switch across phones.** Extend 0027's switch so a member's own payment goes into the shared space with a balance record left behind. Weeks.
3. **Invites, keys and roles.** Key pairs, QR invite, safety number, sealing the space key, the three roles, removing a member with a new key. Server: membership tables. A few weeks.
4. **Sync through the server.** Endpoints to post and fetch sealed changes and snapshots, app integrity checks on each post, retention and deletion, sync when online. Privacy policy 2.6 ships with this stage. A few weeks to a couple of months.
5. **Later, if asked for:** Savings group kind (chama contributions and payouts), a Treasurer role, share rules.

Screens for stages 1 to 3 wait on approved mockups. Each stage gets its own spec.

## Consequences

- One budget across phones becomes possible without our server reading anything.
- The server gains a sixth job and new data (membership), and its cost grows with the number of shared spaces.
- Shared spaces use UUID ids and an operation log; other spaces keep running numbers. Two kinds of id must live side by side.
- Removing a member can stop new changes but cannot take back what their phone already has.
- Losing every member's phone and every key loses the space, unless a vault backup of it exists. The recovery key covers only the person's own backups.

## Open questions for the owner

1. Do you approve option 1, built in the stages above, with the file-only stage first?
2. Which kinds can be shared at first: Household only, or Savings group (chama) too?
3. How many members may a shared space have to start: up to 6, or more for a chama?
4. Should a member see who added each transaction ("added by Achieng"), or only the transaction?
5. Roles: are Owner, Member and Viewer enough, or is a Treasurer needed from the start?
6. Should the server know members' roles to refuse a Viewer's changes, or should only the phones enforce roles (less the server knows, but a modified app could write)?
7. How long may the server keep sealed changes for a phone that never comes back: 90 days?
8. Is a shared space backed up to the vault by each member, by the owner only, or not at all (the other phones being the backup)?
9. Can a person be signed out and still take part, using the file-only stage, or must every member of a synced space have an account?
10. Do you agree the privacy policy needs version 2.6 for stage 4, and a smaller note for stage 1?
