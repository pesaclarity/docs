# 0029. Sending transactions to the person's own Firefly III

Date: 2026-11-21 · Status: Accepted · Builds on [0016](0016-spaces-kept-apart.md), [0018](0018-double-entry-journal.md), [0027](0027-money-between-spaces.md), [0005](0005-pin-certificate-authorities.md) and [0015](0015-gifts-not-a-paywall.md)

The owner approved the direction on 21 November 2026. This record proposes how it works and asks the decisions at the end. Nothing is built yet. Screens wait on an approved mockup.

Firefly III's website and documentation could not be reached while this was written. Every detail of its API below was first written from memory of version 6 and marked **(verify)**. Stage 1 tried each against Firefly III 6.7.9 in Docker on 21 November 2026: details that held are marked **(verified)**, and those that did not are corrected in place. What the server really answered is in the appendix.

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

1. **Server address** and **Personal access token**. Firefly III makes the token under Profile (the person's email at the top), **Remote access and tokens**, Personal Access Tokens (corrected: not Options, Profile, OAuth). It is sent as `Authorization: Bearer <token>` with `Accept: application/vnd.api+json` and `Content-Type: application/json`, on every request, a body or not **(verified)**: without a JSON `Accept`, a refused token answers with a redirect to `/login` instead of 401, and a `POST` with no `Content-Type` is refused with 415. A body is sent with its length: a chunked body did not reach Firefly III behind its own nginx.
2. **Check connection** calls `GET /api/v1/about` **(verified)**. Success shows "Connected to Firefly III 6.7.9" from the version it returns. It also calls `GET /api/v1/about/user` **(verified)** so a token for the wrong user or with no rights fails here, not later. Versions below 6.0 are refused with a plain message.
3. **What to send**: a switch per space, and **Send from**, a date (this month, this year, or everything). Transactions before that date are not sent; each account starts with its balance on that date as its opening balance (section 3).
4. **Start sending.** The first send carries on while the app is open and picks up where it stopped on the next start.

### 2. What becomes what in Firefly III

Firefly III has three kinds of transaction: a withdrawal (asset to expense), a deposit (revenue to asset) and a transfer (asset to asset) **(verified)**. Its accounts are asset, expense, revenue and liability **(verified; a liability is made with `type` `liability` and comes back as `liabilities`)**.

| In Pesa Clarity | In Firefly III |
| --- | --- |
| An account: M-Pesa wallet, bank account, Cash | Asset account, `account_role` `defaultAsset` (Cash: `cashWalletAsset`) **(verified)** |
| A savings account or M-Pesa savings member (0026), Chumz (0028) | Asset account, `account_role` `savingAsset` **(verified)** |
| Fuliza, M-Shwari loan, KCB M-Pesa loan, a debt the person owes | Liability account, `liability_type` `loan` or `debt`, `liability_direction` `debit` (I owe) **(verified; Firefly III shows what is owed as a negative balance)** |
| Money lent to someone | Liability account, `liability_direction` `credit` (owed to me) **(verified)**. Not built yet: the journal has no account for money lent, so it is sent as its role says |
| Held for another space (0027): "Held for Vegetable stall", money on Personal's line that is the business's | Liability account, `liability_type` `debt`, `liability_direction` `debit`: the person holds it for that space, so they owe it. Its balance is the journal's Held for account |
| A balance record (0027): what a payment switched to another space left where its account lives | A deposit from Held for into the account (money in) or a withdrawal from the account to Held for (money out, its fee with it), never income or spending. Its external id is `pesaclarity:<space>:-<record id>`, so it never meets a transaction's |
| The owner's account in a business (0027): "On your M-Pesa" | Asset account, `defaultAsset`, read from the journal's owner's account. The switched payment is sent from the business on it, once |
| `spending`, `fee`, `tax` | Withdrawal to an expense account named after the payee |
| `income` | Deposit from a revenue account named after the payer |
| `ownTransfer`, `saving` | Transfer between the two asset accounts |
| `borrowing` | Deposit from the liability account **(verified)** |
| `debtRepayment` | Withdrawal to the liability account; the fee share (Fuliza, loan fees) is its own withdrawal to the "Fees" expense account **(verified; a transfer to a liability is refused)** |
| `awaitingPayee` | Deposit from a revenue account "Not named yet", tagged `pesaclarity-to-name`; changed when the person names it |
| `reversal` | The opposite type, linked to the original by the notes |
| `correction` | A deposit or withdrawal to "Balance correction" under the category "Balance correction" (corrected: `POST /api/v1/transactions` refuses the `reconciliation` type with 422) |
| Category | `category_name` (Firefly III makes it if new) **(verified)** |
| Payee | The expense or revenue account name, so Firefly III's reports by payee work |
| Space | A tag with the space's name, plus `pesaclarity` (decision 1) |
| Another currency | `currency_code` of the account; `foreign_amount` and `foreign_currency_code` when the money moved in another currency **(verified; required on a transfer between accounts in two currencies)**. Firefly III ships without KES: the app makes it (`POST /api/v1/currencies`) or turns it on before the first account |
| M-Pesa code, message time | `notes`: "M-Pesa SKL4X2Y9ZQ"; the date carries the time with the person's time zone (0010) **(verified)** |

- **Fees.** An M-Pesa or bank fee on a payment is sent as a second withdrawal to the "Fees" expense account, with its own id (`...:fee`). One Firefly III transaction group cannot mix a transfer and a withdrawal **(verified: "All splits must be of the same type.")**, so a transfer with a fee cannot be one group.
- **Accounts first.** Before a transaction is sent, its accounts exist in Firefly III. The app makes each one with `POST /api/v1/accounts` **(verified; it answers 200, not 201, as every create does)** and keeps its Firefly III id. An account already there with the same name and type is used, not made twice **(verified: `GET /api/v1/search/accounts?query=...&field=name&type=...` matches parts of names, so the app checks the exact name)**. Firefly III keeps one account of each kind per name, so a space's own accounts carry its name after Personal's ("M-Pesa · Green Basket Grocers"), and payees are made as expense or revenue accounts and used by id: a name sent as `source_name` was matched to a liability of that name.
- **Not sent:** message text (never kept anyway), balance checkpoints as such, review items that are not yet transactions, goals, budgets, invoices and stock. Reported balances are checked in section 8 instead.

### 3. Opening balances

Each asset account is made with `opening_balance` and `opening_balance_date` **(verified; changing it on a liability needs `liability_type`, `liability_direction`, `interest` and `interest_period` with it, or Firefly III fails with 500)**: its balance in Pesa Clarity's journal at the start of the Send from day. A liability is made with what is owed on that day. When Send from is moved earlier, the opening balances are changed and the older transactions sent.

### 4. No duplicates

- **A stable id on every transaction**: `external_id` = `pesaclarity:<space id>:<transaction id>`, and `:fee` and the like for a transaction sent as several **(verified)**. The space id is the space's own stable id, not its name.
- **A map in the space's database**: Pesa Clarity transaction id to the Firefly III transaction group id of each part (one split per group, so an edit needs no journal id), the `external_id`, the fields last sent and a hash of them. The map is in backups, so a restored phone does not send everything again.
- **Every create** sends `error_if_duplicate_hash: true` **(verified)**. If Firefly III answers that it is a duplicate (422, "Duplicate of transaction #<journal id>." **(verified)**), the app looks the transaction up by its `external_id` (`GET /api/v1/search/transactions?query=external_id_is:"..."`, with the quotes: without them nothing matches **(verified)**) and keeps that id. The hash is of what was first sent and is not changed by an edit, and `external_id` is not unique in Firefly III, so a send that may have arrived (a timeout) looks up its `external_id` before making it again; with no record at all (a first send or a lost record) one search for `external_id_starts:"pesaclarity:<space id>:"` finds what is there. The same lookup rebuilds a lost map.
- **Check** that Firefly III's duplicate hash includes `external_id` **(verified: two identical payments with different `external_id` both arrive)**. Had it not, two real payments with the same amount, payee and day would be refused as duplicates; the app would have sent `error_if_duplicate_hash: false`.
- **An edit** is `PUT /api/v1/transactions/{group id}` **(verified; a single split needs no journal id, the type can change with both accounts, and tags are replaced whole)** with only the fields that changed since the last send. **A delete** is `DELETE /api/v1/transactions/{group id}` **(verified: 204, then 404)**.
- **Switching a transaction to another space** (0027) keeps its Firefly III transaction: the map entry moves with it and the transaction is retagged. If the new space is not being sent, it is deleted from Firefly III. Its `external_id` does not change once sent.
- **Deleted in Firefly III by the person**: an edit that gets 404 is not sent again; the map marks it "removed there". "Send everything again" on the Firefly III screen (behind its menu) repairs on purpose.

### 5. Offline, quietly

- **An outbox in each space's database**: one row per transaction or account waiting to be sent, with what to do (send or delete), when it was queued, attempts and the last error. It is written in the same database write as the change, so nothing is lost if the app stops.
- **The latest wins.** A new change to the same transaction replaces its waiting row. Made and deleted before sending: nothing is sent.
- **A check by hash** catches changes no single edit wrote: a payee renamed or a category changed on many transactions, a migration, a read-again. It compares each transaction's payload hash with the map and queues what differs. It runs after those jobs and on Send now.
- **When it sends**: `FireflyWhenOnline`, the same pattern as `BackupWhenOnline`, after unlock and setup and on each return to the app. One cheap call (`GET /api/v1/about`) first; if that fails, it stops until next time. Then the outbox in order, accounts first, a few at a time with a short pause so a small home server is not flooded. Failed rows wait and back off.
- **No errors elsewhere.** Nothing pops up, and Home and Profile show no warning. The Profile row says "Sending to firefly.example.home · last sent 2 min ago" in grey. The Firefly III screen shows **Last sent**, **Changes waiting**, and a problem when there is one:
  - Can't reach the server: changes wait on the phone.
  - Token refused (401 **(verified)**): sending stops until a new token is saved.
  - Firefly III refused a transaction (422 with a message and the field, such as `transactions.0.amount` **(verified)**): it is shown in a list under the count, with the reason, and skipped; the rest carry on.
- **Turning off** removes the token and the waiting changes. What is already in Firefly III stays. Deleting a space stops sending it; what was sent stays in Firefly III, and the delete screen says so.

### 6. Security

- **The token** is kept in `SecureStore` under one key for the phone, never in a space's database, never in a backup, never in logs or crash reports. It is shown only as dots, with a button to show it while typing.
- **HTTPS is required.** Plain `http://` is allowed only for a local network address (10.x, 172.16-31.x, 192.168.x, `.local`, `.home.arpa`) and only after the person confirms "This connection is not encrypted. Use it only on your home network." (decision 3). iPhone needs the local network permission for this (`NSLocalNetworkUsageDescription` is added; still to try on a phone); Android's rule on plain connections must allow it for these addresses only (the app's network security file says Dart's client ignores it; still to try on a phone). The address check is in Dart and refuses plain http to any other address before a byte is sent.
- **No redirects.** The client does not follow redirects, so the token is never sent to another host. A redirect shows as a problem with the address.
- **Trust.** Our own servers pin certificate authorities (0005). The person's server cannot: we do not know who signs its certificate. So the Firefly III client is its own, separate from both of today's:
  - **By default it uses the phone's trusted authorities, for that one host only.** It is a new `HttpClientFactory.createForUserServer(host)` that refuses any other host, so it can never be used to reach something else with the token.
  - **Option (recommended, decision 2): trust a self-signed certificate after showing its fingerprint once.** Many home servers use a self-signed certificate or their own home authority, which a phone does not trust; Android apps also ignore authorities the person added to the phone, by default (still to try on a phone; Dart loads the system store only). When the handshake fails for that reason, Check connection shows the certificate's SHA-256 fingerprint and asks "Trust this certificate?". If the person says yes, the app keeps that fingerprint for that host and accepts exactly that certificate (`badCertificateCallback` compares it), nothing else. A changed certificate asks again; it is never accepted quietly. This trust is never used for our own hosts.
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

After a send, the screen can compare each asset account's balance in Firefly III (`GET /api/v1/accounts/{id}`, `current_balance` **(verified; a liability the person owes shows negative)**) with Pesa Clarity's. A difference is shown under the count, as a balance check is (0018), never fixed silently. This is also the main test (section 10).

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
6. **Firefly III's own rules.** Recommendation: **let them run when a transaction is made** (`apply_rules: true` **(accepted)**), and on an edit send only the fields Pesa Clarity changed, so a rule's change to another field stays. Alternative: turn rules off for our transactions.
7. **Removed in Firefly III by the person.** Recommendation: **not sent again**, unless they choose "Send everything again". Alternative: Pesa Clarity puts it back on its next edit.
8. **Turning off and deleting a space.** Recommendation: **leave what was sent on the server** and say so. Alternative: offer "Also delete what was sent", which is more calls and can fail half-way.
9. **Transactions not yet named.** Recommendation: **send them** under "Not named yet" with a tag, and change them when named, so Firefly III's balances stay right. Alternative: hold them back until named.
10. **Who gets it.** Recommendation: **every build and everyone**, free, as everything else is (0015). Alternative: Android only first, since iPhone needs the local network permission.
11. **Where it lives.** Recommendation: **a row in Profile under Backup and restore**, opening its own screen. Alternative: inside Backup and restore, which would mix a copy only you can open with data your server reads.


## Appendix: what Firefly III 6.7.9 answered (stage 1)

Tried on 21 November 2026 against `fireflyiii/core:latest` (Firefly III 6.7.9, API 6.7.9, PHP 8.5.12, SQLite, build 2306), run in Docker with a made-up user and a personal access token. The app's tests keep these answers in `test/fixtures/firefly/`.

**Running it.** `docker run -p 8080:8080 -e APP_KEY=<32 characters> -e DB_CONNECTION=sqlite -e APP_URL=http://localhost:8080 -e TRUSTED_PROXIES=** fireflyiii/core`. On a host with no IPv6 its nginx does not start (`listen [::]:8080`); mounting `/etc/nginx/site-opts.d/http.conf.template` without that line fixes it. Docker Hub refused pulls (429) here; `mirror.gcr.io/fireflyiii/core` served the same image. A token for scripts can be made in the container with Laravel's `$user->createToken('name')->accessToken`; `artisan tinker` is not in the image.

| Call | Answer |
| --- | --- |
| `GET /api/v1/about` | 200 `{"data":{"version":"6.7.9","api_version":"6.7.9","php_version":"8.5.12","os":"Linux","driver":"sqlite"}}` |
| `GET /api/v1/about/user` | 200 `data.type` `users`, `attributes.email`, `attributes.role` `owner` |
| Any call, wrong token, JSON `Accept` | 401 `{"message":"Unauthenticated.","exception":"AuthenticationException"}` |
| Any call, wrong token, no JSON `Accept`; `/` and `/api/v1/about/` | 302 to `/login`: never follow, the token would go along |
| `POST` with no `Content-Type` | 415 "Content-Type header cannot be empty." |
| `POST /api/v1/accounts` asset in KES on a new install | 422 "The selected currency code is invalid.": KES does not exist. `POST /api/v1/currencies {code,name,symbol,decimal_places,enabled}` makes it (the symbol must be unique); `POST /api/v1/currencies/{code}/enable` turns one on. An account in a currency that exists but is off is accepted |
| `POST /api/v1/accounts` asset | 200; `account_role` `defaultAsset`, `cashWalletAsset`, `savingAsset`; `opening_balance` "0" leaves none |
| `POST /api/v1/accounts` liability `loan`, `debit`, opening 300 | 200; `type` `liabilities`, `opening_balance` "-300", `current_balance` "-300.00"; with `credit`, positive |
| Same name and type twice | 422 "This account name is already in use."; an asset and a liability, or an asset and an expense account, may share a name |
| `GET /api/v1/search/accounts?query=Fuliz&field=name&type=liabilities` | Finds "Fuliza": parts of names match; without `type`, the "Initial balance for" accounts come too |
| `POST /api/v1/transactions` withdrawal, deposit, transfer | 200; `external_id`, `tags`, `category_name` (made if new), `notes`, `currency_code` kept; date offset kept (`2026-11-03T14:05:00+03:00`); amounts come back as `"450.000000000000"` |
| Transfer KES to a USD account without `foreign_amount` | 422 on `transactions.0.foreign_amount`; with `foreign_amount` and `foreign_currency_code` it moves KES out and USD in |
| Deposit from a liability; withdrawal to one; withdrawal from one; deposit into one | All accepted: borrowing, repaying, a charge added to what is owed, and what is owed going down. A transfer to a liability is refused |
| `type` `reconciliation` | 422 "Created zero transaction journals." (and "No error yet." with a destination) |
| Group with a transfer and a withdrawal | 422 "All splits must be of the same type." |
| Same payload again, `error_if_duplicate_hash: true` | 422 `{"message":"Duplicate of transaction #8.","errors":{"transactions.0.description":[...]}}` (#8 is a journal id) |
| Same payload, another `external_id` | 200: the hash includes `external_id` |
| Same `external_id` after an edit to the first | 200: the hash is of the first payload; `external_id` is not unique |
| `GET /api/v1/search/transactions?query=external_id_is:"pesaclarity:sp1:1"` | The one group. Without quotes: nothing. `external_id_starts:"..."` lists by prefix, with `meta.pagination` |
| `PUT /api/v1/transactions/{id}` with one split, no journal id | 200; only the fields sent change; `type` with `source_id` and `destination_id` changes the kind; `tags` replace the old ones |
| `DELETE /api/v1/transactions/{id}` | 204; again 404 `{"message":"Resource not found",...}`; `PUT` on it 404 |
| `PUT /api/v1/accounts/{id}` opening balance | 200 on an asset; on a liability without `liability_direction` (and type, interest) 500 "Undefined array key" |
| `GET /api/v1/accounts/{id}` | `current_balance` as a string; a liability owed shows negative |
| Personal access tokens in the web interface | Profile, Remote access and tokens |

**Money between spaces (decision 0027), added after stage 1.** A payment switched from Personal to a business is sent as the two journals hold it: the business sends the sale, a deposit into the owner's account; Personal sends its balance record, a deposit from "Held for <business>" into M-Pesa. Firefly III's net worth counts the money once (the asset in each space, less the liability in Personal), income counts once (the business's), and every account ends at its balance in Pesa Clarity: the app's tests check this against a stand-in for the API that keeps balances as Firefly III does (`test/features/firefly/firefly_sender_test.dart`). Not yet tried against a real Firefly III: the integration test does it when `FIREFLY_URL` and `FIREFLY_TOKEN` are set. A liability of type `debt` with `debit` is the same call as the loans above, which were verified.

Not tried here: the iPhone local network permission and Android's rule on plain connections (they need a phone), and a self-signed certificate on a real home server (the app's tests use a local server with a test certificate).
