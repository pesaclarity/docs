# Pesa Clarity: product spec and designs

2026-10-26 · Dan Njuguna

> Snapshot of the live spec doc (https://claude.ai/artifact/WLbTbhV4YuMDB94NKTiHLU) taken 26 Oct 2026, with the design-system table marking the red as Pesa Clarity's own. Screen designs live in the [design canvas](https://claude.ai/artifact/XARkzWeYgZH91hqYd7KvPJ). Changes to the spec go through pull requests here from now on.

Pesa Clarity is a free, ad-free Android app that turns M-Pesa and bank messages into a clear daily plan: what you can spend today, what is due, where money leaks, and when you will be debt-free. All analysis runs on the phone; the server only signs users in, holds backups it cannot read, and combines federated model updates.

## Principles

Five commitments shape every decision in this spec.

1. **Free forever, no ads.** Every feature is free for every user; no premium tier, no paywalls, no advertising.
2. **Your data stays yours, on your phone.** Parsing, analysis and model training run on the device. The server keeps only sign-up, sign-in and sign-out, encrypted backups it cannot read, and federated model updates.
3. **Plain answers, not spreadsheets.** Seven screens replace a 17-tab workbook; every number taps through to the transactions behind it.
4. **Export in two taps.** Any view exports to CSV, Excel or PDF; a full backup or workbook is always one menu away.
5. **Honest money maths.** Own-account transfers never count as spending, every import reconciles to its statement, and informal family debts count as real debts.

## Problem and users

M-Pesa users can see every transaction but not their finances: building a budget sheet from two months of real statements surfaced five problems.

- **Spending is invisible.** Over half of bank outflows moved to M-Pesa, where 1,262 transactions in 55 days hid what money was actually spent on.
- **Leaks go unnoticed.** Lifestyle, friends and family transfers reached 17–27% of income in a month before anyone tagged them.
- **Cash runs out before payday.** Balances near zero mid-month caused failed card charges and about 150 overdraft (Fuliza) draws in two months.
- **Debts are informal.** Half the debts were family loans with no record, rate or term.
- **Income is irregular.** Foreign-currency invoices land on varying dates at the bank's rate.

| Segment | Pain | What wins them |
| --- | --- | --- |
| Salaried, M-Pesa heavy (22–40, 50k–300k KES a month) | Salary gone by mid-month | Daily safe-to-spend, leak alerts, payday plan |
| Freelancers and remote workers | Irregular, foreign income | Invoices, multi-currency, tax reserve |
| Young families | Rent, school fees, family support | Recurring bills, set-asides, reminders |
| People in debt | Bank, Fuliza and family loans tangled | Debt plan with a debt-free date |

Out of scope for v1: businesses, chamas and joint household accounts.

## Design

Five bottom tabs answer five questions, with debts and profile one tap deeper; the editable source is the [design canvas](https://claude.ai/artifact/XARkzWeYgZH91hqYd7KvPJ). Figures shown are illustrative.

| Tab | Answers | Replaces sheet tabs |
| --- | --- | --- |
| Home | How am I doing today? | Dashboard, Accounts |
| Activity | Where did my money go? | Ledger, Explorer, Payees |
| Plan | What's coming, and is it covered? | Budget, Recurring, Budget history |
| Goals | Am I getting there? (opens Debts) | Debts, Goals, Progress |
| Insights | What should I change? | Leakages, charts |

### Home

Safe to spend today (weekday or weekend limit), plan used, left this month, runway, Financial Health Index, upcoming bills, and the + button to add or import.

### Activity

Search, account and type filters, export, weekly in and out, and a day-grouped feed. Own transfers show as not spending; unknown numbers prompt the user to name the payee.

### Plan

Month switcher, planned vs expected income split by debt, tax, living and school, category status (Done, In progress, Over), and the surplus split.

### Goals

Debt-free date and progress, debts at a glance, goal rings, and net worth with its monthly change.

### Debts

Remaining balance, payoff strategy (avalanche, snowball, custom), this payday's split, a Fuliza tracker, and a what-if slider.

### Insights

- **What changed:** up to three findings for the month so far against the same days of last month: the category that moved most, fees, and money in.
- **Spending:** week, month or year; spending by category as a ring, and so far against the period before. The year adds spending month by month.
- **Leakage:** total, share of income and change on last month, with what goes to lifestyle, friends, family and fees.
- **Financial Health Index:** the score, the four pillars and what helps most. A pillar opens its two indicators and what brings each to full marks.
- **Tags** and a one-page PDF report.

Every finding, category and month opens Activity filtered to the transactions behind it. Before any transactions, one card points to My accounts and statement import.

### Profile

Account details, my accounts (each account's currency, and the institutions whose messages the app reads), base currency, timezone, subscriptions, payday, import, export and backup, security, sign out and delete account.

### Spaces

Profile › Spaces lists every space, where the app opens, Recently deleted and Delete all my data. Data is never shared between spaces ([0016](../decisions/0016-spaces-kept-apart.md)).

- **A space's settings:** name; kind (any but Personal, after a warning of what comes in and goes away); what a business sells (products, services or both); and for the open space its accounts, base currency, day end, receipts and invoices, tax and backup.
- **Transactions** are never edited, only recategorised or moved. Long-press in Activity to choose several, then Move to another space, choosing the category there or matching each one; they leave the first space.
- **Clear data** removes all of a space's records, or only a date range's transactions, sales and quick records, and keeps its name, accounts, categories and settings. **Delete space** removes it, after its name is typed; Personal cannot be deleted on its own.
- Both offer a backup first, an optional 30-day keep (restored from **Recently deleted**) and, only if ticked, deleting that space's vault and Drive backups.
- **Delete all my data** asks for the app lock PIN, can also delete online backups and the sign-in, then erases everything on the phone, the database key included.

### Support Pesa Clarity

Every feature is free; gifts unlock nothing ([0015](../decisions/0015-gifts-not-a-paywall.md)). Only the F-Droid build asks: a Support screen in Profile and a Home card under the red card that never shows in the first 30 days, then for 3 days at most every 60, never when money is tight and never within six months of a gift. One-time gifts of 50, 100, 200, 500 or any amount open the giving page in the browser. A gift's own M-Pesa message brings a thank-you and a list of the person's gifts, on the phone only. Play and iPhone show nothing; giving is on the website's Sponsor page.

### Subscriptions

What repeats and what it costs a month and a year. Repeat charges found in messages (same payee, a month apart, similar amount) are offered to track or mark as a one-off; tracked ones follow their latest charge, flag price changes, and are reminded before they fall due.

### Currency

Setup asks which currency the person counts in, suggesting their phone region's; Profile switches it. Transactions keep the currency they were paid in and are converted at the rates of their day (kept from the first fetch each day), the rate the person got, or a rate they set; plan, goals, debts and subscriptions are in the base currency. Lists show each amount in its own currency first with the base amount under it, each amount keeps the currency its message prints, and accounts at the same bank are told apart by their masked account numbers, so a KES and a USD account stay separate; past amounts are fixed from the messages on the phone. Accounts added by hand have a currency the person picks for new amounts. A currency converter in Profile › Base currency converts between any two currencies. See [0009](../decisions/0009-base-currency.md) and [0014](../decisions/0014-rates-of-the-day.md).

### Backup and restore

Last backup status, destinations switched on independently (vault, own Google Drive, phone or SD card), Back up now and Share a copy, and restore from any source with the passphrase or recovery key.

### Onboarding: welcome and sign-up

The promise up front, phone number with SMS code, Google, or no account at all. The iPhone version adds Sign in with Apple, as App Store rules require.

### Onboarding: my accounts

The user ticks the institutions they transact with; each tick activates a parser pack. No logins. "Read messages from" sets how far back the first scan reads: every message (the default), the last 3, 6 or 12 months, or from a date; it can be changed later in Profile, and a wider choice reads the older messages on the next scan. The main button starts that scan behind the 60-second first insight.

### Privacy

Plain-language promises, the three consent switches (improve the app on, licensed models off, crash reports off), the full policy, and Email me a copy.

### iPhone: connect M-Pesa

Connect your messages: three steps, once (open Shortcuts › Automation › Message; pick MPESA and the bank as senders, Run Immediately; add "Add transaction from message" with the message and sender), an Open Shortcuts button, and the status: "Connected · Last message 2 min ago" or "Waiting for your first message". Statement import fills any gaps.

The iPhone design follows iOS rather than the Android one: large titles, a full-width tab bar, inset grouped lists with coloured icons in Profile, the sliding segmented control, date wheels, ticked option lists and the system font, with figures in Space Grotesk. Boards are in the design canvas under "iPhone".

### Design system

| Token | Value | Use |
| --- | --- | --- |
| Background | #FBF8F8 | Screen ground |
| Surface | #FFFFFF, border #EFE6E7 | Cards, lists, nav bar |
| Text | #1C1215, muted #6F6064 | Body, labels |
| Brand (Pesa Red) | #C8102E | Hero cards, active tab, primary buttons |
| Link red | #B81D2F | Links and alerts on white |
| Success | #059669 (text: #047857) | Income, on track, cleared |
| Warning | #B45309 | In progress |
| Type | Space Grotesk for numbers and titles; Inter for text | |
| Shape | 20–28 px card radius, 44 px minimum touch target, floating pill nav | |

Full brand guide, logo files and tokens: [BRAND.md](../BRAND.md).

## MVP scope

The Android MVP ships ten features; everything else waits until beta users ask for it.

| Feature | What it does |
| --- | --- |
| SMS capture | Reads bank and M-Pesa messages on the phone through per-institution parser packs |
| Statement import | M-Pesa and bank PDFs (password-protected too) for history; reconciles to the closing balance |
| Payees | Name a number or till once; its category and leak type apply to every past and future transaction |
| Safe to spend | Daily limit split weekday vs weekend, from what is left of the day-to-day budget |
| Plan | Recurring bills plus one-offs per month; surplus split to debt and savings; unpaid bills carried forward |
| Debts | All debt types, payoff strategies, payday split, debt-free date (details below) |
| Goals | Targets, monthly amount needed, progress |
| Insights | Leakages, trends, fees, Financial Health Index |
| Reminders | Local notifications before bills, card charges and payday tasks |
| Export and backup | CSV, Excel, PDF from any screen; full backup zip; encrypted cloud backup |

**Tags and categories are separate.** A category says what kind of spending a transaction is (one per transaction, drives budgets); tags say what it is part of (#roadtrip, #wedding, #reimbursable, any number per transaction). Tags work with or without a category; uncategorised transactions still count in totals as Uncategorised; new categories can be created inline from a transaction; tags filter Activity, total on Insights and appear in exports. The app ships a default Kenyan category set so nobody starts empty; Eating out and Snacks are separate, and so are Loan repayment and Fuliza repayment.

**Categories are applied automatically when the app is confident.** The person's own earlier choice for the payee always applies; message-type rules and well-known payees apply at 60% confidence or more; weaker guesses stay one-tap suggestions. An automatic category shows as Auto, is stored apart from the person's choice, and never overrides it.

### What makes it win

Features that set it apart, split by when they ship. Value in the first minute decides whether people keep the app, so the first insight leads.

| Feature | Ships | Why it wins |
| --- | --- | --- |
| 60-second first insight | MVP | On setup, scans the chosen period (every message by default) and shows income, spending, leakages and overdraft fees before any configuration |
| Cash-flow forecast | MVP | "At this pace you hit zero on the 18th" and "your card charge on the 20th will fail": prevents overdrafts instead of reporting them |
| Payday autopilot | MVP | When the salary message lands, proposes the split (rent, debts, set-asides, savings) as a checklist |
| Lent and borrowed | MVP | Who owes whom among friends and family, with a polite WhatsApp reminder |
| Runs on cheap phones | MVP | Small download, works on 2 GB RAM Android Go phones, light on data |
| Open-source parser packs | After beta | Community adds banks, SACCOs and wallets; coverage grows faster than one developer could manage |
| Swahili, Sheng-friendly wording | After beta | Reaches users beyond English-first professionals |
| Optional spending caps | After beta | User-set limits ("weekend lifestyle max 3,000") with a gentle nudge at 80% |
| Freelancer tax helper | After beta | eTIMS invoices, withholding certificates and the iTax deadline |
| Your own cash-flow report | After beta | A PDF the user chooses to share with a lender or landlord |
| Monthly money story | After beta | Private recap with an optional shareable card showing no amounts |

Left out on purpose: bank logins or scraping, in-app payments, crypto, investment advice, social or competitive features.

Phase 2: invoices and timesheets, multi-currency income at actual rates, tax reserve estimate, iOS. Out of scope: bank logins or screen scraping, payments from the app, investment advice.

## Getting data in

SMS is the live source on Android; statements backfill history and verify balances; manual entry covers cash. How statements are read, including passwords and per-bank statement packs, is in [Statement import](statement-import.md).

**My accounts, not account linking.** During setup the user ticks the institutions they actually transact with (M-Pesa, I&M, KCB, Equity, Airtel Money and so on). Each tick activates that institution's parser pack and adds its sender IDs to the watch list; the app then scans past messages from those senders and listens for new ones. There are no bank logins, no credentials and no connections to any institution's systems; accounts can be added or switched off any time in Profile.

Google Play allows SMS access under its "SMS-based money management" exception, after a Permissions Declaration reviewed by a person; it is approved only when SMS budgeting is the app's core function ([Google Play policy](https://support.google.com/googleplay/android-developer/answer/9047303)).

### SMS parser engine

| Part | What it does |
| --- | --- |
| Sender filter | Reads only the senders of accounts the user selected (from MPESA, I&M, KCB, Equity, StanChart, Co-op, Airtel Money and more); every other message is ignored and never stored |
| Parser pack per sender | Regex rules with named groups (receipt, amount, direction, counterparty, account, own account, currency, balance, fee, time, and for a debt such as Fuliza the total owed and its due date), one per message type: sent, received, paybill, till, Fuliza, Fuliza repayment, reversal, bank. Sender IDs match whatever their case |
| Test suite | Every rule ships with anonymised samples and expected fields; a pack releases only when all pass |
| Signed updates | New packs download as signed rule files, without an app update |
| Review inbox | Unmatched messages from known senders wait on Home as "N messages to check". The on-phone message reader suggests amount, payee, date, balance and receipt for the person to confirm or change; "Read messages like this the same way" saves the layout so later ones are added during capture. The reader trains on synthetic messages only |
| Dedupe | Receipt number, amount and time match SMS against later statement imports |

### Rules that keep the numbers honest

- **Own transfers are neutral.** Bank to own M-Pesa, M-Pesa to bank, money market and savings moves are linked pairs, never spending; the real spend is counted once, where it happens.
- **Reconcile every import** to the statement's closing balance and show any gap.
- **Collapse noise:** one line per day for Fuliza draws and repayments, money market round-ups, charges and airtime.
- **Fuliza fees** are repayments minus draws, shown as a fee.
- **Money received from people** stays neutral until the user names the payee (gift, loan, repayment).
- **Balances come from the messages.** Each account's balance is the last balance a message, statement or the person reported, plus what happened since. A balance the message reader suggested or the person typed while checking a message only counts once it adds up.
- **Fuliza is a debt on M-Pesa.** When M-Pesa pays out more than the wallet holds, Fuliza covers the rest and that is owed; the next money in repays it first. Any M-Pesa balance above zero means nothing is owed. Fuliza owed comes off On hand.
- **Gaps are explained, not hidden.** Profile › Balances shows each account and whether it matches its messages. Where two balances don't agree, the balance check shows where it went (the day's entries, messages waiting to be checked, likely the missing one) and offers "Add what's missing" as a real transaction, or a visible correction.
- **Money without messages.** Add an account for cash, and Set the balance to what an account really holds; the difference is a balance correction in Activity, never income or spending, and can be undone.

### iOS capture via Shortcuts

iOS lets no app read SMS, so the iPhone app captures messages through Apple's Shortcuts automations, with statement import as the safety net. Processing stays on the phone. Built; see decision 0011.

| Piece | Who does it | Effort |
| --- | --- | --- |
| "Add transaction from message" App Intent | Built into the app; appears in Shortcuts automatically. Takes the message and its sender, runs without opening the app | None |
| Message automation (Message, senders MPESA and the bank, Run Immediately, the action above) | User, guided by Connect your messages | About a minute, once |
| Connection check | App shows "Connected · Last message 2 min ago", or "Waiting for your first message" | None |

Apple does not let apps create automations, so the trigger step stays manual. The action queues each message in a file protected until the iPhone's first unlock, so messages that land while it is locked still count. The app reads the queue like an inbox (same parser, review inbox and deduplication) and drops each message once read, or after 35 days unread. Still to confirm on a device: that current iOS runs message automations without asking each time.

#### iOS onboarding flow

1. **Sign in:** email with a one-time code, or use without an account. Sign in with Apple comes later; it needs a server endpoint.
2. **Currency and My accounts:** as on Android.
3. **Connect:** Connect your messages, in place of the Android SMS scan. "Do this later" carries on; Profile links back to it.
4. **Backfill:** import M-Pesa and bank statement PDFs from Profile for the history wanted.
5. **Live:** new messages arrive through the automation; receipt numbers deduplicate any overlap with statements, which can lag a day or two.
6. **Catch-up:** a monthly reminder to import the latest statement fills any messages the automation missed; reconciliation shows any gap.

#### App Store requirements

| Requirement | How the app meets it |
| --- | --- |
| Sign in with Apple | Required only if third-party sign-in is offered; the iPhone app has email sign-in only for now, so Apple sign-in comes with Google sign-in on iPhone |
| In-app account deletion | Profile > Delete account |
| Privacy nutrition labels | Declares sign-in details and encrypted backups only; no tracking |
| Purpose of statement files | Explained in the import screen: used on the phone, never uploaded |
| Encryption export compliance | Declared in App Store Connect; standard encryption, normally exempt |
| No private APIs | Messages reach the app only through Shortcuts and user-shared files |

Apple review should be simpler than Google's SMS permission review, since nothing needs a special entitlement.

## Debt management

Debt management promises one thing: a realistic debt-free date, and a payday plan that gets there.

| Feature | What it does |
| --- | --- |
| Every debt type | Bank loans, cards, overdrafts (Fuliza is M-Pesa's overdraft), salary advances, mobile loans, SACCO loans, family and friend loans without interest |
| Auto-detection | SMS and statements spot repayments and suggest which debt they belong to |
| Strategies | Avalanche (highest rate first), snowball (smallest first) or custom; each shows the debt-free date and total interest |
| Payday split | Minimums first, the rest to the target debt, on each payday |
| Family loans | Agreed due dates, part payments, gentle reminders, no interest maths |
| Fuliza tracker | Draws, fees and days overdrawn per month, with a nudge when usage climbs |
| What-ifs | "Add 5,000 a month" or "clear the overdraft first" recalculates instantly |
| Progress | Paid, remaining, debts cleared, interest saved vs minimums only |

Fuliza is treated as an overdraft, like a bank or card overdraft: its daily charges are overdraft interest, shown today as Fuliza fees. Open: rename the category to Overdraft repayment and the fees to Overdraft fees once bank and card overdrafts are parsed the same way.

## Financial Health Index

A 0–100 score, recalculated monthly on the phone, sums eight indicators across Spend, Save, Borrow and Plan, adapting the Financial Health Network's framework to the user's own data rather than a survey.

| Pillar | Indicator | Full marks (12.5 points) when |
| --- | --- | --- |
| Spend | Spending vs income | Spend under 90% of income |
| Spend | Bills on time | Every recurring bill paid by its due date |
| Save | Emergency cushion | 3+ months of essentials saved |
| Save | Savings rate | 20%+ of income saved or invested |
| Borrow | Debt load | Debt payments under 20% of income |
| Borrow | Overdraft reliance | No Fuliza or overdraft use in the month |
| Plan | Leakage share | Leakages under 10% of income |
| Plan | Plan adherence | Categories within budget |

Bands: Struggling 0–39, Coping 40–69, Healthy 70–100. Home shows the score and its change; tapping it names the weakest pillar and the single action that would raise it most. Thresholds are starting points to tune with beta users.

## Architecture, privacy and data model

The phone does all the work and holds all the data; the server has three narrow jobs and never sees a transaction.

### On the phone

- **App:** Flutter (decided: its own rendering engine keeps the custom design identical on every phone, isolates handle heavy background parsing, and Drift with SQLCipher is mature), Android first; SQLite via Drift, encrypted with SQLCipher; charts with fl_chart.
- **Engine:** SMS and PDF parsing, categorisation, budget, safe-to-spend, leakages, debt payoff, Financial Health Index and reminders, all offline.
- **Models:** a small categoriser (TensorFlow Lite) fine-tunes on the user's own confirmations and trains federated updates on spending patterns.

### On the server

| Service | Stores | Never stores |
| --- | --- | --- |
| Auth | Account id, sign-in contact, sign-up, sign-in and sign-out events | Any financial data |
| Backup vault | Backups encrypted on the phone (AES-256-GCM, key from the user's passphrase via Argon2id) | Keys or readable data |
| Federated learning | Clipped, noised model updates combined by secure aggregation | Raw data or individual updates in the clear |

Infra (decided, [0004](../decisions/0004-eu-hosting-and-email-sign-in.md)): in the EU, the API on Google Cloud Run, Postgres on Neon, backup ciphertext on Cloudflare R2 (EU jurisdiction) and sign-in emails through Brevo; all fit free tiers at proof-of-concept traffic. Checks run on the developer's machine before each push (`tool/check`); GitHub Actions builds, releases and deploys. Signed parser-pack and model releases tracked in MLflow.

### Privacy and compliance

- **Kenya Data Protection Act 2019:** register with the ODPC; file a data protection impact assessment for federated training; honour access and deletion requests within 30 days.
- **Lost passphrase = lost backup:** offer a printable recovery key at setup.
- **Statement passwords** are used once in memory, then discarded; PDFs never leave the phone. See [Statement import](statement-import.md#passwords).
- **App lock:** PIN or biometric, auto-lock, hide balances on open, remote sign-out of devices.
- **Hiding amounts:** one eye, the same on every screen, hides every amount across the app; it starts from "Hide balances on open". It sits at the right of a screen's title, or on the red card a screen leads with. The counter (Sell, Pay, tabs, Close the day, the record sheets), forms where an amount is typed, and receipts, invoices and statements chosen to share or print always show amounts.
- **No analytics:** product decisions rely on interviews and opt-in feedback; an opt-in, anonymous crash report is the one exception.

### Mobile security

Target OWASP MASVS level 2 (the standard for apps handling financial data), verified by an external penetration test before public launch.

| Layer | Control |
| --- | --- |
| Data at rest | SQLCipher database; its key generated and held in the Android Keystore (hardware-backed, StrongBox where available), released only after the user unlocks |
| Unlock | BiometricPrompt (strong class) with PIN fallback; increasing delays after failed attempts; auto-lock when the app goes to the background |
| Screen | FLAG_SECURE blocks screenshots, screen recording and previews in the app switcher; hide-balances mode |
| SMS handling | Only selected senders are read; raw message text is discarded after parsing, keeping fields and a hash for dedupe |
| Permissions | SMS, notifications, restoring reminders after a restart, and the network (sign-in, backups, the privacy policy) only; no contacts, location or broad storage access. A test fails the build if the list changes |
| OS backups | Android auto-backup off, so the database never lands in Google Drive; only the app's own end-to-end encrypted backup |
| Network | TLS with cleartext off. Our servers are pinned to their certificate authorities, not their keys, because the hosts renew certificates with new keys every few months ([0005](../decisions/0005-pin-certificate-authorities.md)); Google Drive uses the phone's own authorities. No third-party SDKs or trackers |
| Device integrity | A light root and tamper check warns once in Profile and never blocks use. Play builds attach a Play Integrity token, bound to the request, to backup uploads (and later model updates); the server refuses a token that fails. F-Droid builds cannot attest, so uploads without a token follow a server setting, set to allow |
| Code | R8 and Flutter obfuscation; no secrets in the app; dependency scanning (OSV-Scanner) in `tool/check` before each push |
| Updates | Parser packs and models signed (Ed25519) and verified on the phone before use |
| Sessions | Short-lived tokens, refresh token in the Keystore, device list with remote sign-out |
| Lost phone | Data stays encrypted and locked; remote sign-out cuts backup and sync access; restore on a new phone with the passphrase or recovery key |

### Backup and restore: the user chooses where

Every backup is the same file, encrypted on the phone with the user's passphrase before it goes anywhere, so the destination never matters to security; the user picks where it lives and can restore from any of them.

| Destination | How | Notes |
| --- | --- | --- |
| Pesa Clarity vault | Automatic, daily or weekly | Default; the server stores ciphertext only |
| Your Google Drive | User signs in to their own Drive; app writes to its own folder (drive.file scope) | Never touches our server |
| Phone or SD card | Saves an encrypted .pcbackup file | Works fully offline |
| Any app via share sheet | Email, WhatsApp to self, Dropbox, OneDrive, USB transfer | Manual |
| Later: Dropbox, OneDrive, Nextcloud/WebDAV direct | Scheduled backups to those services | Phase 2 |

Restore: on a new phone, choose the source, pick a backup by date, enter the passphrase or recovery key; the app checks the file's integrity before replacing anything and keeps the current data until the restore succeeds. Users can keep several destinations at once (for example vault plus a monthly copy on the SD card) and see the date of the last successful backup in Profile.

### Sign-up, consent and the privacy policy

| Moment | What happens |
| --- | --- |
| Welcome | Email with a one-time code, Continue with Google (Play build only), Sign in with Apple (iPhone, later), or Use without an account; a link to the Privacy Policy. No SMS codes: they cost too much for an open-source project |
| First run | Plain-language privacy summary and three choices: improve the app (federated updates, on), contribute to licensed models (off), crash reports (off) |
| Any time | Profile > Privacy shows the same summary and switches, the full policy, and its version and date |
| Email me a copy | Sends the full policy as text plus PDF to the Google account's email, or to an address the user types; a typed address is used only for that send and then discarded |
| Policy changes | Version number bumps; on material changes the app shows what changed in plain words and asks again before new uses start |

The policy itself must cover, per the Data Protection Act: the controller and ODPC registration, what is collected (sign-in details, encrypted backups, model updates), what never leaves the phone, legal bases, retention, rights (access, correction, deletion, objection to automated decisions), cross-border hosting, and a contact for requests. Draft it in plain English, then have a Kenyan data protection lawyer review it before launch.

The Data Protection Officer for the Pesa Clarity platform is Dan Njuguna (njugunahdann@gmail.com), named in the policy (from version 1.7) and on the website as the contact for requests.

### Data model (on the phone)

| Table | Key fields |
| --- | --- |
| accounts | name, type (bank, M-Pesa, SACCO, MMF, cash), currency, opening balance and date |
| transactions | account, time, amount, direction, currency, rate, payee, item, category override, goal or debt, receipt no., source |
| payees | raw identifiers (masked phone, till, paybill and account), display name, default category, leak type |
| categories | name, type (income, expense, debt, savings, tax, transfer), daily-spending flag; tags in their own table, linked many-to-many to transactions |
| recurring | name, category, amount, frequency, start, end, account |
| budget_items and snapshots | one-offs and carried-forward arrears; frozen month-end results |
| debts | lender, type, starting balance, rate, minimum, planned, formal or family |
| goals | target, currency, date, priority, status |
| subscriptions | name, payee, amount, currency, frequency, next date, tracking or one-off |
| imports and parser_packs | file hash, period, reconciliation result; pack versions and test status |
| journal_entries and postings | double-entry journal ([0018](../decisions/0018-double-entry-journal.md)): each transaction, opening and balance gap as balanced postings. Stage 1: kept up to date underneath, but nothing reads it yet |

## Federated learning and model licensing

The app earns nothing from its users. The only possible income is licensing federated models that describe patterns, never people, built only from users who opt in.

### Two training tracks

| Track | Purpose | Default |
| --- | --- | --- |
| App track | Improves parsing, categorisation and insights for users | On, can be switched off |
| Commercial track | Trains models that may be licensed | Off; separate opt-in, withdraw any time |

### What could be licensed

| Model | Who would license it | Risk to users |
| --- | --- | --- |
| SMS and statement parser plus categoriser for Kenyan institutions | Banks, SACCOs, fintechs categorising their own customers' transactions | Low: patterns of messages and merchants |
| Merchant and category directory | Accounting and expense tools | Low |
| Aggregate spending trends (groups of 1,000+, differentially private) | Researchers, retailers, planners | Low to medium |
| Credit or risk scoring | Lenders | High: profiling with legal effects; not offered |

### Release conditions

- DP-SGD with a published privacy budget, secure aggregation and minimum cohort sizes on every release. The budget covers one person, not one transaction, and counts every training run and audit on the same data. The noise is never reproducible from a published seed.
- A membership-inference audit passes before any model leaves the server.
- A data protection impact assessment filed with the ODPC; legal review by a Kenyan data protection lawyer before the first licence.
- Any licence income goes first to keeping the app free; until then, running costs come from grants or the founder.

## Open source

Everything is open source: the app, the server, the parser packs and this spec. Open code is how users and auditors verify that money data never leaves the phone.

| Part | Licence | Why |
| --- | --- | --- |
| Mobile app and server | AGPL-3.0, plus a commercial licence on request | Forks, including hosted ones, must stay open; companies that cannot accept AGPL can buy a commercial licence, which can fund the project |
| Parser packs and test fixtures | Apache-2.0 | Maximum adoption and community contributions; banks and fintechs can use them freely |
| App-track model weights (shipped in the app) | CC BY-SA 4.0 | Free like the app, so anyone can check what runs on their phone and F-Droid can ship it |
| Commercial-track model weights | Written licence only, never published | Built only from people who opted in to the commercial track; terms forbid credit scoring, profiling and re-identification, keeping model licensing possible |
| Docs and designs | CC BY 4.0 | Free to reuse with credit |

What open source requires:

- **Two build flavours.** A Play Store build (Google Sign-In, Play Integrity) and a FOSS build for F-Droid with no Google services; email sign-in and backups work in both. F-Droid accepts no prebuilt native libraries, so its build compiles SQLCipher from source and leaves out PDFium, which means no statement PDF import there for now ([0006](../decisions/0006-fdroid-build-without-prebuilt-libraries.md)).
- **Reproducible builds,** so anyone can confirm the Play Store app matches the public source.
- **Synthetic test data only.** Every SMS sample in the repo is generated or fully anonymised; never commit real messages.
- **Protected keys.** App signing, parser-pack signing and server secrets stay outside the repo, in a hardware key or secrets manager.
- **Trademark the name.** The code can be forked, but the name and logo stay protected, so scam clones cannot pose as the real app and harvest SMS.
- **Contributor terms.** A contributor licence agreement (or copyright assignment) keeps the commercial licence option legally possible; a code of conduct, contributing guide, and security policy with a private disclosure address.
- **Public roadmap and issues** on GitHub; parser packs accept community pull requests for new banks, SACCOs and wallets, each gated by its test suite.

### Repositories

All code lives under the GitHub organisation `pesaclarity`. All seven repositories exist from the start; each is built in the phase shown. See [REPOS.md](../REPOS.md) for the full table.

| Repo | Codename | Licence |
| --- | --- | --- |
| `.github` | | CC BY 4.0 |
| `docs` | | CC BY 4.0 (name and logo excluded) |
| `ujumbe-parsers` | ujumbe (message) | Apache-2.0 |
| `pesaclarity-app` | | AGPL-3.0 + commercial on request |
| `pesaclarity-web` | dirisha (window) | AGPL-3.0 (code), CC BY 4.0 (text) |
| `pesaclarity-backend` | ngome (fortress) | AGPL-3.0 + commercial on request |
| `pesaclarity-ml` | akili (intelligence) | AGPL-3.0 (code); app-track weights CC BY-SA 4.0, commercial-track weights by written licence |

`pesaclarity-app` depends on `ujumbe-parsers` (Dart package `ujumbe`), so parser packs can be versioned, tested and contributed independently of app releases.

## Roadmap

Working evenings, a solo developer can reach a private beta in about 14 weeks, with each phase gated on one result.

| Phase | Weeks | Scope | Gate |
| --- | --- | --- | --- |
| 0. Validate | 1–2 | Interview 15 target users with the screen designs | 10 of 15 would grant SMS access |
| 1. Parsers and engine | 3–6 | M-Pesa, I&M and KCB parser packs with tests; statement import; local ledger, plan and safe-to-spend | Two months of real data reconcile to the cent |
| 2. MVP app | 7–11 | Seven screens, debts, goals, insights, FHI, reminders, export, encrypted backup | 5 testers use it weekly for 3 weeks |
| 3. Play review and beta | 12–14 | Permissions Declaration, ODPC registration, 30–50 beta users, federated app track | SMS exception approved; 40% four-week retention |
| 4. Grow | 15+ | More banks, invoices, multi-currency, iOS, commercial track pilot | Funding secured from a grant or first model licence |

## Risks and open questions

The two risks that could stop the project are Google refusing SMS access and the cost of keeping parsers current; both have fallbacks.

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Play refuses the SMS exception | No live capture | Make SMS budgeting the core flow in the declaration; fall back to statement import and the notification listener |
| Institutions change message wording | Silent misses | Test suites per pack, review inbox, signed pack updates within days |
| Users stop engaging | App goes stale | Daily safe-to-spend notification, payday plan, monthly report |
| Federated models leak information | Privacy harm, legal exposure | DP-SGD, secure aggregation, cohort minimums, audits; no credit scoring |
| No revenue for a long time | Founder burnout | Tiny server costs; grant applications around the beta |

Open questions:

- ~~Brand red~~ — resolved: #C8102E is Pesa Clarity's own red (see BRAND.md)
- Product name: Pesa Clarity is the working name; decide after the proof of concept
- Which banks after M-Pesa, Equity, I&M, KCB and Standard Chartered (deposits and card payments; its other messages go to review)? Loop and Airtel Money are review-only until their wording is known (decision 0013). A Co-operative Bank pack is in ujumbe-parsers but not yet in the app. Whether its rule for money received on M-Pesa counts money twice, and which senders are real, is open in ujumbe-parsers #44.
- Flutter or React Native: decided, Flutter (26 Oct 2026)
- Which grant programmes to apply to for the beta?
