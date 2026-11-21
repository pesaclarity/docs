# 0029. Sending transactions to the person's own Firefly III

Date: 2026-11-21 · Status: Accepted · Builds on [0016](0016-spaces-kept-apart.md), [0018](0018-double-entry-journal.md), [0027](0027-money-between-spaces.md), [0005](0005-pin-certificate-authorities.md) and [0015](0015-gifts-not-a-paywall.md)

The owner approved the direction on 21 November 2026. This record proposes how it works and asks the decisions at the end. Nothing is built yet. Screens wait on an approved mockup.

Firefly III's website and documentation could not be reached while this was written. Every detail of its API below is from memory of version 6 and is marked **(verify)**: check it against a running Firefly III before building (stage 1).

## Context

[Firefly III](https://github.com/firefly-iii/firefly-iii) is a free, open-source personal finance manager that people run on their own server, often at home. It is double-entry, like Pesa Clarity's journal (0018), and has a REST API. Some people who use Pesa Clarity already run it. They want what Pesa Clarity reads from M-Pesa and bank messages to land there too, without typing it again or importing a CSV every month.

What exists today (`pesaclarity-app` `origin/dev`):

- **Nothing leaves the phone with a transaction in it**, except an encrypted backup the person sets up. The privacy policy (2.4) says our server "never sees a transaction". This must stay true.
- **Each space has its own encrypted database** (`space_databases.dart`) and is kept apart (0016). A transaction lives in one space; it can be switched to another, and the first space then keeps only a balance record (0027).
- **The journal** gives every transaction a `LedgerRole` (`spending`, `income`, `fee`, `tax`, `ownTransfer`, `borrowing`, `debtRepayment`, `saving`, `awaitingPayee`, `reversal`, `correction`). Accounts, payees, categories, debts and goals are rows in the space's database. `Transactions.id` is an auto-increment number, unique only inside its space.
- **Work that waits for a connection** already has a pattern: `BackupWhenOnline` calls `BackupController.catchUp()` after unlock and on every return to the app. It shows nothing, and offline it tries again next time.
- **Network clients** come from `HttpClientFactory` (`pinned_http_client.dart`). `create()` trusts only our pinned roots and only our hosts (0005). `createUnpinned()` uses the phone's authorities, for services such as Google Drive.
- **Secrets** go in `SecureStore` (Android Keystore, iPhone Keychain). It is not part of a backup.
- **Export** (CSV, Excel, PDF) already exists. It is a one-off file, not a connection.

## Decision

Pesa Clarity can **send** a space's transactions, one way, to a Firefly III server the person runs. It is **off** until the person turns it on in Profile. Data goes only to that server, never to Pesa Clarity's servers. Pesa Clarity stays the source: it creates, changes and deletes what it sent. Nothing is read back. Two-way sync is a later stage with its own conflict rules (section 9).

### 1. Setting it up

A new row in Profile, **Firefly III**, under Backup and restore. It opens the Firefly III screen.

1. **Server address** and **Personal access token**. Firefly III makes the token under Options, Profile, OAuth **(verify)**. It is sent as `Authorization: Bearer <token>` with `Accept: application/vnd.api+json` and `Content-Type: application/json` **(verify)**.
2. **Check connection** calls `GET /api/v1/about` **(verify)**. Success shows "Connected to Firefly III 6.1.24" from the version it returns. It also calls `GET /api/v1/about/user` **(verify)** so a token for the wrong user or with no rights fails here, not later. Versions below 6.0 are refused with a plain message.
3. **What to send**: a switch per space, and **Send from**, a date (this month, this year, or everything). Transactions before that date are not sent; each account starts with its balance on that date as its opening balance (section 3).
4. **Start sending.** The first send carries on while the app is open and picks up where it stopped on the next start.

### 2. What becomes what in Firefly III

Firefly III has three kinds of transaction: a withdrawal (asset to expense), a deposit (revenue to asset) and a transfer (asset to asset) **(verify)**. Its accounts are asset, expense, revenue and liability **(verify)**.

| In Pesa Clarity | In Firefly III |
| --- | --- |
| An account: M-Pesa wallet, bank account, Cash | Asset account, `account_role` `defaultAsset` (Cash: `cashWalletAsset`) **(verify)** |
| A savings account or M-Pesa savings member (0026), Chumz (0028) | Asset account, `account_role` `savingAsset` **(verify)** |
| Fuliza, M-Shwari loan, KCB M-Pesa loan, a debt the person owes | Liability account, `liability_type` `loan` or `debt`, `liability_direction` `debit` (I owe) **(verify)** |
| Money lent to someone | Liability account, `liability_direction` `credit` (owed to me) **(verify)** |
| `spending`, `fee`, `tax` | Withdrawal to an expense account named after the payee |
| `income` | Deposit from a revenue account named after the payer |
| `ownTransfer`, `saving` | Transfer between the two asset accounts |
| `borrowing` | Deposit from the liability account **(verify)** |
| `debtRepayment` | Withdrawal to the liability account; the fee share (Fuliza, loan fees) is its own withdrawal to the "Fees" expense account **(verify)** |
| `awaitingPayee` | Deposit from a revenue account "Not named yet", tagged `pesaclarity-to-name`; changed when the person names it |
| `reversal` | The opposite type, linked to the original by the notes |
| `correction` | Firefly III's own reconciliation type if the API accepts it **(verify)**; otherwise a deposit or withdrawal under the category "Balance correction" |
| Category | `category_name` (Firefly III makes it if new) **(verify)** |
| Payee | The expense or revenue account name, so Firefly III's reports by payee work |
| Space | A tag with the space's name, plus `pesaclarity` (decision 1) |
| Another currency | `currency_code` of the account; `foreign_amount` and `foreign_currency_code` when the money moved in another currency **(verify)** |
| M-Pesa code, message time | `notes`: "M-Pesa SKL4X2Y9ZQ"; the date carries the time with the person's time zone (0010) **(verify)** |

- **Fees.** An M-Pesa or bank fee on a payment is sent as a second withdrawal to the "Fees" expense account, with its own id (`...:fee`). One Firefly III transaction group cannot mix a transfer and a withdrawal **(verify)**, so a transfer with a fee cannot be one group.
- **Accounts first.** Before a transaction is sent, its accounts exist in Firefly III. The app makes each one with `POST /api/v1/accounts` **(verify)** and keeps its Firefly III id. An account already there with the same name and type is used, not made twice **(verify the search: `GET /api/v1/search/accounts?query=...&field=name`)**.
- **Not sent:** message text (never kept anyway), balance checkpoints as such, review items that are not yet transactions, goals, budgets, invoices and stock. Reported balances are checked in section 8 instead.

### 3. Opening balances

Each asset account is made with `opening_balance` and `opening_balance_date` **(verify)**: its balance in Pesa Clarity's journal at the start of the Send from day. A liability is made with what is owed on that day. When Send from is moved earlier, the opening balances are changed and the older transactions sent.

### 4. No duplicates

- **A stable id on every transaction**: `external_id` = `pesaclarity:<space id>:<transaction id>` **(verify the field)**. The space id is the space's own stable id, not its name.
- **A map in the space's database**: Pesa Clarity transaction id to Firefly III transaction group id and journal id, the `external_id`, and a hash of what was last sent. The map is in backups, so a restored phone does not send everything again.
- **Every create** sends `error_if_duplicate_hash: true` **(verify)**. If Firefly III answers that it is a duplicate (422 **(verify)**), the app looks the transaction up by its `external_id` (`GET /api/v1/search/transactions?query=external_id_is:"..."` or the form the running server accepts **(verify)**) and keeps that id. The same lookup rebuilds a lost map.
- **Check** that Firefly III's duplicate hash includes `external_id` **(verify)**. If it does not, two real payments with the same amount, payee and day would be refused as duplicates; then the app sends `error_if_duplicate_hash: false` and relies on its own map and lookup.
- **An edit** is `PUT /api/v1/transactions/{group id}` **(verify)** with only the fields Pesa Clarity owns. **A delete** is `DELETE /api/v1/transactions/{group id}` **(verify)**.
- **Switching a transaction to another space** (0027) keeps its Firefly III transaction: the map entry moves with it and the transaction is retagged. If the new space is not being sent, it is deleted from Firefly III. Its `external_id` does not change once sent.
- **Deleted in Firefly III by the person**: an edit that gets 404 is not sent again; the map marks it "removed there". "Send everything again" on the Firefly III screen (behind its menu) repairs on purpose.

### 5. Offline, quietly

- **An outbox in each space's database**: one row per transaction or account waiting to be sent, with what to do (send or delete), when it was queued, attempts and the last error. It is written in the same database write as the change, so nothing is lost if the app stops.
- **The latest wins.** A new change to the same transaction replaces its waiting row. Made and deleted before sending: nothing is sent.
- **A check by hash** catches changes no single edit wrote: a payee renamed or a category changed on many transactions, a migration, a read-again. It compares each transaction's payload hash with the map and queues what differs. It runs after those jobs and on Send now.
- **When it sends**: `FireflyWhenOnline`, the same pattern as `BackupWhenOnline`, after unlock and setup and on each return to the app. One cheap call (`GET /api/v1/about`) first; if that fails, it stops until next time. Then the outbox in order, accounts first, a few at a time with a short pause so a small home server is not flooded. Failed rows wait and back off.
- **No errors elsewhere.** Nothing pops up, and Home and Profile show no warning. The Profile row says "Sending to firefly.example.home · last sent 2 min ago" in grey. The Firefly III screen shows **Last sent**, **Changes waiting**, and a problem when there is one:
  - Can't reach the server: changes wait on the phone.
  - Token refused (401 **(verify)**): sending stops until a new token is saved.
  - Firefly III refused a transaction (422 with a message **(verify)**): it is shown in a list under the count, with the reason, and skipped; the rest carry on.
- **Turning off** removes the token and the waiting changes. What is already in Firefly III stays. Deleting a space stops sending it; what was sent stays in Firefly III, and the delete screen says so.

### 6. Security

- **The token** is kept in `SecureStore` under one key for the phone, never in a space's database, never in a backup, never in logs or crash reports. It is shown only as dots, with a button to show it while typing.
- **HTTPS is required.** Plain `http://` is allowed only for a local network address (10.x, 172.16-31.x, 192.168.x, `.local`, `.home.arpa`) and only after the person confirms "This connection is not encrypted. Use it only on your home network." (decision 3). iPhone needs the local network permission for this **(verify)**; Android's rule on plain connections must allow it for these addresses only **(verify how dart:io follows it)**.
- **No redirects.** The client does not follow redirects, so the token is never sent to another host. A redirect shows as a problem with the address.
- **Trust.** Our own servers pin certificate authorities (0005). The person's server cannot: we do not know who signs its certificate. So the Firefly III client is its own, separate from both of today's:
  - **By default it uses the phone's trusted authorities, for that one host only.** It is a new `HttpClientFactory.createForUserServer(host)` that refuses any other host, so it can never be used to reach something else with the token.
  - **Option (recommended, decision 2): trust a self-signed certificate after showing its fingerprint once.** Many home servers use a self-signed certificate or their own home authority, which a phone does not trust; Android apps also ignore authorities the person added to the phone, by default **(verify for dart:io)**. When the handshake fails for that reason, Check connection shows the certificate's SHA-256 fingerprint and asks "Trust this certificate?". If the person says yes, the app keeps that fingerprint for that host and accepts exactly that certificate (`badCertificateCallback` compares it), nothing else. A changed certificate asks again; it is never accepted quietly. This trust is never used for our own hosts.
- **What it can reach.** The app's network rules are not widened for anything but the one host the person typed.

### 7. Privacy

This is the first time Pesa Clarity sends transactions anywhere but an encrypted backup. It does so only when the person turns it on, only to the server they typed, and never through us. The policy and website must say so before it ships.

Changes needed (not made here):

- **Privacy policy** (`pesaclarity-web` `src/pages/privacy.astro`, version 2.4 to 2.5): in "The short version", a sentence that if you turn on Firefly III sync, your transactions are sent to the Firefly III server you choose, and never to ours. A new section "Firefly III" saying what is sent (section 2), that it is off until you turn it on, that the token stays on the phone and out of backups, that turning it off stops sending and leaves what was sent on your server, and that we cannot see or delete data on your server. "Your choices" gains it as off by default. "Keeping it safe" says it goes over an encrypted connection unless you choose a plain local one.
- **Security page** (`src/pages/security.astro`) and **docs** (`src/content/docs/security/pinned-certificate-authorities.md`): your own server is trusted by the phone's authorities for that host only, or by a fingerprint you confirmed; it is not pinned.
- **Help**: `src/content/help/privacy-controls/your-privacy-choices.md` adds Firefly III to what can leave the phone; a new help article "Send to Firefly III" (setup, the token, the certificate question, what each problem means).
- **Features page** (`src/pages/features.astro`): one line under privacy or export.
- **Store labels.** Data sent to a server the person runs is not collected by us; check that the Play data safety form and Apple's privacy label still need no change, and record why in the release notes **(verify the current store guidance)**.
- **Product spec**: the Privacy section of `spec/PRODUCT-SPEC.md` gains the same sentence.

### 8. Checking it adds up

After a send, the screen can compare each asset account's balance in Firefly III (`GET /api/v1/accounts/{id}`, `current_balance` **(verify)**) with Pesa Clarity's. A difference is shown under the count, as a balance check is (0018), never fixed silently. This is also the main test (section 10).

### 9. Later: two-way

Not in this decision. Reading changes back from Firefly III needs rules for which side wins, a way to tell our writes from the person's, and probably Firefly III's webhooks. It waits until one-way has been used.

### 10. Stages and tests

| Stage | Work |
| --- | --- |
| 1 | Verify the API. Run Firefly III 6 in Docker (`fireflyiii/core` with SQLite or MariaDB), make a token, and try every call marked (verify) with `curl`. Record the real requests and answers as fixtures in the app's test folder. This needs network access to Docker Hub (`registry-1.docker.io`, `auth.docker.io`, `production.cloudflare.docker.com`) on the machine that runs it. Update this record where the API differs. |
| 2 | Mockup approved (Profile row, Firefly III screen, problems, certificate, turning off). |
| 3 | Core, no screens: the mapping from journal to Firefly III payloads, the outbox and map tables (a migration in each space's database), the hash check, the client with its own trust rule, the sender with back-off. |
| 4 | Screens and `FireflyWhenOnline`. Privacy policy and website changes ready to publish with the release. |
| 5 | A beta with people who run Firefly III, then release in every build (Play, F-Droid, iPhone). |

Tests:

- **Mapping**: one test per `LedgerRole`, fee, currency and liability case, against the recorded payloads from stage 1.
- **Outbox**: latest wins; make-then-delete sends nothing; a crash between write and send loses nothing; switching space retags; a space turned off stops its rows.
- **No duplicates**: a duplicate answer adopts the existing id; a lost map is rebuilt by `external_id`; a restore sends nothing twice; two real identical payments both arrive.
- **Problems**: offline, 401, 422, 404 on edit, 5xx and a timeout, each with the right state on screen and nothing shown elsewhere.
- **Security**: the token never appears in a backup file, a log or a crash report; a redirect is refused; the client refuses any other host; a self-signed certificate is refused until confirmed, accepted after, and refused again when it changes; `http://` to a public address is refused.
- **Integration** (a separate CI job, not on every pull request): start Firefly III in Docker, run the app's sender against it with a month of fixture transactions across two spaces (M-Pesa, Fuliza, a bank, Cash, KES and USD), then check that every account's balance matches Pesa Clarity's, edits and deletes arrive, and running it twice changes nothing. The CI runner needs Docker Hub access.

## Alternatives

- **Export a file the person imports with Firefly III's Data Importer.** No connection and no token. But it is by hand every time, edits and deletes do not follow, and duplicates are the importer's problem. Export stays for that.
- **Send through Pesa Clarity's server.** Would work when the home server is only reachable from the internet. But transactions would pass through us, which breaks the policy's promise. Rejected.
- **Two-way sync now.** More useful to some, but needs conflict rules and doubles the risk of damage to someone's books. Later (section 9).
- **Pin the person's certificate always.** Safest, but a server with a normal public certificate (Let's Encrypt) renews every 60 to 90 days and would ask each time. Rejected as the default; it is the self-signed option.
- **Trust any certificate on a local address.** Easy, but anyone on the same Wi-Fi could take the token. Rejected.

## Consequences

- People who run Firefly III get their M-Pesa and bank money there without typing it, and keep Pesa Clarity's reading of messages.
- Transactions can now leave the phone, but only when the person turns it on and only to their own server. The policy, website and help must change first.
- Each space's database gains three tables (outbox, map of transactions, map of accounts) and a migration.
- A third kind of network client (the person's server) sits beside our pinned one and the phone's; it must never be reused for anything else.
- Firefly III's API can change between versions. The integration job catches it; the app refuses versions below 6.0.
- What was sent stays on the person's server after turning off or deleting a space. We cannot remove it, and say so.

## Owner's decisions

The owner accepted every recommendation below on 21 November 2026.

1. **A space in Firefly III.** Recommendation: **one Firefly III user, each space a tag** (plus `pesaclarity`). It works with 0027, where a business sale can sit on the owner's Personal M-Pesa account: in one user that is one asset account with the business's tag. The alternative, **a separate Firefly III user per space**, keeps spaces apart as 0016 does on the phone and gives each its own reports, but needs a token per space, and a transaction switched between spaces would have to be deleted in one user and made in the other, with its account copied there.
2. **The person's certificate.** Recommendation: **the phone's authorities for that host only, plus trusting a self-signed certificate after showing its fingerprint once** (asked again if it changes). Alternative: the phone's authorities only, which is simpler but leaves out many home servers.
3. **Plain http on a home network.** Recommendation: **allow it only for local addresses, after a one-time warning**. Alternative: HTTPS only.
4. **Send from.** Recommendation: **default to 1 January this year**, with opening balances on that day. Alternatives: this month, or everything.
5. **Fees.** Recommendation: **a separate withdrawal to "Fees"** for every fee, so totals match Pesa Clarity. Alternative: add it to the payment's amount, which is simpler but hides fees.
6. **Firefly III's own rules.** Recommendation: **let them run when a transaction is made** (`apply_rules: true` **(verify)**), and on an edit send only the fields Pesa Clarity changed, so a rule's change to another field stays. Alternative: turn rules off for our transactions.
7. **Removed in Firefly III by the person.** Recommendation: **not sent again**, unless they choose "Send everything again". Alternative: Pesa Clarity puts it back on its next edit.
8. **Turning off and deleting a space.** Recommendation: **leave what was sent on the server** and say so. Alternative: offer "Also delete what was sent", which is more calls and can fail half-way.
9. **Transactions not yet named.** Recommendation: **send them** under "Not named yet" with a tag, and change them when named, so Firefly III's balances stay right. Alternative: hold them back until named.
10. **Who gets it.** Recommendation: **every build and everyone**, free, as everything else is (0015). Alternative: Android only first, since iPhone needs the local network permission.
11. **Where it lives.** Recommendation: **a row in Profile under Backup and restore**, opening its own screen. Alternative: inside Backup and restore, which would mix a copy only you can open with data your server reads.
