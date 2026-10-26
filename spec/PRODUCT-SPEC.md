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

Leakage total and share of income, where it goes (lifestyle, friends, family, fees), six-month trend, and a one-page PDF report.

### Profile

Account details, my accounts (the institutions whose messages the app reads), payday and currency, import, export and backup, security, sign out and delete account.

### Backup and restore

Last backup status, destinations switched on independently (vault, own Google Drive, phone or SD card), Back up now and Share a copy, and restore from any source with the passphrase or recovery key.

### Onboarding: welcome and sign-up

The promise up front, phone number with SMS code, Google, or no account at all. The iPhone version adds Sign in with Apple, as App Store rules require.

### Onboarding: my accounts

The user ticks the institutions they transact with; each tick activates a parser pack. No logins. The main button starts the 90-day scan behind the 60-second first insight.

### Privacy

Plain-language promises, the three consent switches (improve the app on, licensed models off, crash reports off), the full policy, and Email me a copy.

### iPhone: connect M-Pesa

Three steps: add the shortcut (one tap), create the message automation with the guided steps, then wait for confirmation. Statement import fills any gaps.

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

**Tags and categories are separate.** A category says what kind of spending a transaction is (one per transaction, drives budgets); tags say what it is part of (#roadtrip, #wedding, #reimbursable, any number per transaction). Tags work with or without a category; uncategorised transactions still count in totals as Uncategorised; new categories can be created inline from a transaction; tags filter Activity, total on Insights and appear in exports. The app ships a default Kenyan category set so nobody starts empty.

### What makes it win

Features that set it apart, split by when they ship. Value in the first minute decides whether people keep the app, so the first insight leads.

| Feature | Ships | Why it wins |
| --- | --- | --- |
| 60-second first insight | MVP | On setup, scans the last 90 days and shows income, spending, leakages and overdraft fees before any configuration |
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

SMS is the live source on Android; statements backfill history and verify balances; manual entry covers cash.

**My accounts, not account linking.** During setup the user ticks the institutions they actually transact with (M-Pesa, I&M, KCB, Equity, Airtel Money and so on). Each tick activates that institution's parser pack and adds its sender IDs to the watch list; the app then scans past messages from those senders and listens for new ones. There are no bank logins, no credentials and no connections to any institution's systems; accounts can be added or switched off any time in Profile.

Google Play allows SMS access under its "SMS-based money management" exception, after a Permissions Declaration reviewed by a person; it is approved only when SMS budgeting is the app's core function ([Google Play policy](https://support.google.com/googleplay/android-developer/answer/9047303)).

### SMS parser engine

| Part | What it does |
| --- | --- |
| Sender filter | Reads only the senders of accounts the user selected (from MPESA, I&M, KCB, Equity, Co-op, NCBA, Airtel Money and more); every other message is ignored and never stored |
| Parser pack per sender | Regex rules with named groups (receipt, amount, direction, counterparty, account, balance, fee, time), one per message type: sent, received, paybill, till, Fuliza, reversal |
| Test suite | Every rule ships with anonymised samples and expected fields; a pack releases only when all pass |
| Signed updates | New packs download as signed rule files, without an app update |
| Review inbox | Unmatched messages from known senders wait for the user to tag fields once, creating a local rule |
| Dedupe | Receipt number, amount and time match SMS against later statement imports |

### Rules that keep the numbers honest

- **Own transfers are neutral.** Bank to own M-Pesa, M-Pesa to bank, money market and savings moves are linked pairs, never spending; the real spend is counted once, where it happens.
- **Reconcile every import** to the statement's closing balance and show any gap.
- **Collapse noise:** one line per day for Fuliza draws and repayments, money market round-ups, charges and airtime.
- **Fuliza fees** are repayments minus draws, shown as a fee.
- **Money received from people** stays neutral until the user names the payee (gift, loan, repayment).

### iOS capture via Shortcuts

iOS lets no app read SMS, so the iOS version (after the Android MVP) captures messages through Apple's Shortcuts automations, with statement import as the safety net. Processing stays on the phone.

| Piece | Who does it | Effort |
| --- | --- | --- |
| "Add transaction from message" App Intent | Built into the app; appears in Shortcuts automatically | None |
| Ready-made Pesa Clarity shortcut | User taps a link and "Add Shortcut" | One tap, once |
| Message automation per sender (Message, Sender: MPESA, Run Immediately, choose the shortcut) | User, guided step by step from My accounts | About 30 seconds per institution |
| Connection check | App shows "Connected, last message 2 min ago" once messages arrive, or "Not receiving yet" | None |

Apple does not let apps create automations, so the trigger step stays manual. One shortcut serves every institution; only the trigger repeats per sender. Before building, confirm the current iOS version still runs message automations without asking each time.

#### iOS onboarding flow

1. **Sign in:** phone number with SMS code, Sign in with Apple, or Continue with Google; or use without an account.
2. **My accounts:** tick the institutions used.
3. **Backfill:** share M-Pesa and bank statement PDFs covering the history wanted, up to today; the app shows the 60-second first insight.
4. **Connect:** add the Pesa Clarity shortcut (one tap), then create one message automation per institution with the guided setup.
5. **Live:** new messages arrive through the automation; receipt numbers deduplicate any overlap with statements, which can lag a day or two.
6. **Catch-up:** a monthly reminder to import the latest statement fills any messages the automation missed; reconciliation shows any gap.

#### App Store requirements

| Requirement | How the app meets it |
| --- | --- |
| Sign in with Apple | Offered on iPhone alongside Google sign-in, as Apple requires when third-party sign-in exists |
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
| Every debt type | Bank loans, cards, overdrafts, salary advances, Fuliza, mobile loans, SACCO loans, family and friend loans without interest |
| Auto-detection | SMS and statements spot repayments and suggest which debt they belong to |
| Strategies | Avalanche (highest rate first), snowball (smallest first) or custom; each shows the debt-free date and total interest |
| Payday split | Minimums first, the rest to the target debt, on each payday |
| Family loans | Agreed due dates, part payments, gentle reminders, no interest maths |
| Fuliza tracker | Draws, fees and days overdrawn per month, with a nudge when usage climbs |
| What-ifs | "Add 5,000 a month" or "clear the overdraft first" recalculates instantly |
| Progress | Paid, remaining, debts cleared, interest saved vs minimums only |

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

Infra: Docker on a small VPS or Cloud Run, GitHub Actions CI, signed parser-pack and model releases tracked in MLflow.

### Privacy and compliance

- **Kenya Data Protection Act 2019:** register with the ODPC; file a data protection impact assessment for federated training; honour access and deletion requests within 30 days.
- **Lost passphrase = lost backup:** offer a printable recovery key at setup.
- **Statement passwords** are used once in memory, then discarded; PDFs never leave the phone.
- **App lock:** PIN or biometric, auto-lock, hide balances on open, remote sign-out of devices.
- **No analytics:** product decisions rely on interviews and opt-in feedback; an opt-in, anonymous crash report is the one exception.

### Mobile security

Target OWASP MASVS level 2 (the standard for apps handling financial data), verified by an external penetration test before public launch.

| Layer | Control |
| --- | --- |
| Data at rest | SQLCipher database; its key generated and held in the Android Keystore (hardware-backed, StrongBox where available), released only after the user unlocks |
| Unlock | BiometricPrompt (strong class) with PIN fallback; increasing delays after failed attempts; auto-lock when the app goes to the background |
| Screen | FLAG_SECURE blocks screenshots, screen recording and previews in the app switcher; hide-balances mode |
| SMS handling | Only selected senders are read; raw message text is discarded after parsing, keeping fields and a hash for dedupe |
| Permissions | SMS and notifications only; no contacts, location or broad storage access |
| OS backups | Android auto-backup off, so the database never lands in Google Drive; only the app's own end-to-end encrypted backup |
| Network | TLS 1.3 with certificate pinning to the three server endpoints; no third-party SDKs or trackers |
| Device integrity | Play Integrity API: warn on rooted or tampered devices; the server accepts backups and model updates only from genuine app installs |
| Code | R8 and Flutter obfuscation; no secrets in the app; dependency scanning in CI |
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
| Welcome | Phone number with SMS code, Continue with Google, Sign in with Apple (iPhone), or Use without an account; links to Terms and Privacy Policy |
| First run | Plain-language privacy summary and three choices: improve the app (federated updates, on), contribute to licensed models (off), crash reports (off) |
| Any time | Profile > Privacy shows the same summary and switches, the full policy, and its version and date |
| Email me a copy | Sends the full policy as text plus PDF to the Google account's email, or to an address the user types; a typed address is used only for that send and then discarded |
| Policy changes | Version number bumps; on material changes the app shows what changed in plain words and asks again before new uses start |

The policy itself must cover, per the Data Protection Act: the controller and ODPC registration, what is collected (sign-in details, encrypted backups, model updates), what never leaves the phone, legal bases, retention, rights (access, correction, deletion, objection to automated decisions), cross-border hosting, and a contact for requests. Draft it in plain English, then have a Kenyan data protection lawyer review it before launch.

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
| imports and parser_packs | file hash, period, reconciliation result; pack versions and test status |

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

- DP-SGD with a published privacy budget, secure aggregation and minimum cohort sizes on every release.
- A membership-inference audit passes before any model leaves the server.
- A data protection impact assessment filed with the ODPC; legal review by a Kenyan data protection lawyer before the first licence.
- Any licence income goes first to keeping the app free; until then, running costs come from grants or the founder.

## Open source

Everything is open source: the app, the server, the parser packs and this spec. Open code is how users and auditors verify that money data never leaves the phone.

| Part | Licence | Why |
| --- | --- | --- |
| Mobile app and server | AGPL-3.0, plus a commercial licence on request | Forks, including hosted ones, must stay open; companies that cannot accept AGPL can buy a commercial licence, which can fund the project |
| Parser packs and test fixtures | Apache-2.0 | Maximum adoption and community contributions; banks and fintechs can use them freely |
| Trained federated model weights | Separate model licence | Code is open, but weights built from opted-in users can be licensed, keeping model licensing possible |
| Docs and designs | CC BY 4.0 | Free to reuse with credit |

What open source requires:

- **Two build flavours.** A Play Store build (Google Sign-In, Play Integrity) and a FOSS build for F-Droid with no Google services; phone sign-in and local backup work in both.
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
| `pesaclarity-ml` | akili (intelligence) | AGPL-3.0 (code); weights under a separate model licence |

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
- Which two banks after M-Pesa, I&M and KCB?
- Flutter or React Native: decided, Flutter (26 Oct 2026)
- Which grant programmes to apply to for the beta?
