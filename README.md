<img src="brand/logo-wordmark.svg" alt="Pesa Clarity" width="360">

# Pesa Clarity docs

Pesa Clarity is a free, ad-free Android app, with an iPhone version in the works, that turns M-Pesa and bank messages into a clear daily plan for you, your home and your business: what you can spend today, what is due, where money leaks, and when you will be debt-free. All analysis runs on the phone; the server only signs users in, holds backups it cannot read, and combines federated model updates.

This repo is the source of truth for what we build and how it looks.

## What the app does today

**Your money** ([spec](spec/PRODUCT-SPEC.md), [income for everyone](spec/income-for-everyone.md), [statement import](spec/statement-import.md)):
- reads M-Pesa, Airtel Money and bank SMS on the phone, and imports statements for past months;
- safe to spend today, or a daily amount until the next money for gigs, family support and seasonal income;
- a payday plan, week by week, a rainy day fund and goals;
- debts with a debt-free date and a Fuliza tracker, bills and reminders, subscriptions;
- a money health score and insights, with every figure tappable;
- a base currency with public exchange rates;
- a message reader for new wording, and categories that learn on the phone;
- encrypted backups per space, and app lock.

**A space for each part of life** ([spec](spec/income-for-everyone.md)): Personal, Business, Household, Savings group, Rental property, Farm and Something else. Each has its own records, categories and Home.

**For a business** ([business money](spec/business-money.md)):
- Today with Sell, Bought, Got paid and Paid, and Pay with a receipt;
- Money (the cash book), People (customers who owe and suppliers owed, with statements), and reminders with messages the owner can change;
- stock with restocks and counts, close the day, and pay yourself into Personal;
- proper books underneath, in everyday words;
- insights, full records and reports in the owner's chosen style;
- tax estimates from a tax pack that ships with the app;
- receipts, invoices, hours, attached documents and the owner's M-PESA QR code;
- a fit for products, services or both.

**Your look:** theme (System, Light or Dark), text fields underlined or boxed, and the style of reports, receipts and invoices.

**What makes it different:**
- on the phone and offline;
- private by design;
- free, ad-free and open source;
- plain English;
- made for Kenyan money;
- a download of about 19 MB.

| Doc | What's in it |
| --- | --- |
| [spec/PRODUCT-SPEC.md](spec/PRODUCT-SPEC.md) | Product spec: principles, users, screens, MVP, data capture, debts, Financial Health Index, architecture, privacy, open source, roadmap, risks |
| [BRAND.md](BRAND.md) | Name, voice, logo, colour, type |
| [REPOS.md](REPOS.md) | The organisation's repositories, codenames, licences, and build phase |
| [decisions/](decisions/) | Architecture decision records |
| [research/INTERVIEW-GUIDE.md](research/INTERVIEW-GUIDE.md) | Phase 0 user interviews: who, script, scoring and the go/no-go gate |
| [brand/](brand/) | Logo files and design tokens |

## Principles

1. **Free forever, no ads.**
2. **Your data stays yours, on your phone.**
3. **Plain answers, not spreadsheets.**
4. **Export in two taps.**
5. **Honest money maths.**

## Changing the docs

Open a pull request. Decisions that change the spec get a short note in the PR description saying what changed and why.

## Get involved

- **Star [the app](https://github.com/pesaclarity/pesaclarity-app)** so more people find a private money app that runs on their phone.
- Help with the spec, user research and decisions is welcome: open an issue to discuss a change first.
- **Pick a first issue:** [good first issues](https://github.com/pesaclarity/docs/issues?q=is%3Aopen+label%3A%22good+first+issue%22) or [help wanted](https://github.com/pesaclarity/docs/issues?q=is%3Aopen+label%3A%22help+wanted%22).
- **Read [CONTRIBUTING.md](CONTRIBUTING.md)** before your first pull request. Never include real messages, statements or phone numbers.

## Licence

Text and designs: [CC BY 4.0](LICENSE). The Pesa Clarity name and logo are trademarks and are not covered by that licence (see [BRAND.md](BRAND.md#licence)).
