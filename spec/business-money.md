# Business: money owed and daily sales

**Status:** Proposed · **Date:** 2026-11-05 · **Mockups:** "Business: money owed and daily sales" on the app screens canvas · **Related:** [Spaces](income-for-everyone.md#5-spaces-personal-and-business-stage-3), pesaclarity/pesaclarity-app#149

## Why

A business space today shows profit this month, sales and costs by category, and what the owner paid themselves. It answers "did money come in?". Small shop owners also need to know two other things every day:

1. **Who owes me, and whom do I owe?** Credit sales are the main reason dukas keep a paper book.
2. **What did I sell today, and is my cash right?** Cash sales leave no message, so today they are invisible.

Both work fully offline. They need no new permissions, no server and no privacy policy change.

## Who it is for

Business spaces only, at first. The same parts could later serve farms (credit to buyers, cash at market) and rental property (tenants who owe rent).

## 1. Money owed

### What people see

- **Business Home:** an "Owed to you" row (total, number of customers, how many are due today) and a "You owe" row (total, next due date). Tapping either opens Money owed.
- **Money owed screen:** two tabs, *Owed to you* and *You owe*. Each person shows the amount left, what it was for and when it is due, with a Remind button. A "Paid this month" list underneath.
- **Sold on credit sheet:** customer, amount, what for (optional), pay by (today, in a week, or a date), and phone for reminders (optional). Opened from "+ On credit" on Home or from Money owed.
- **Remind sheet:** a ready message the owner can edit, sent by WhatsApp or SMS. The message names the business and the amount and, when set, the till or Paybill number to pay to.
- **Paid by M-Pesa:** when a payment arrives from someone who owes money, Home asks "Otieno paid 1,200 by M-Pesa. Mark his credit as paid?". A part payment comes off the balance. Nothing is marked paid without the owner's tap.
- **You owe:** suppliers and others the business owes, with a due date and an optional reminder on the owner's own phone.

### How it works

- **Owed to you** builds on `LentLoans` and `LoanPayments`, which already track money lent and part payments. A credit sale is a lent loan with a customer, an optional phone and an optional note.
- **You owe** builds on `Debts` (informal debts with instalments), with the supplier as the lender.
- Both tables already live in each space's own database, so a business's customers never show in Personal.
- **Matching payments:** an incoming payment is matched to a customer by the sender name or number in the message, the same way payees are matched today. The match is only offered, never applied on its own.
- **Reminders** open the phone's WhatsApp or SMS app with the message filled in (`wa.me` link or `sms:` intent). The app sends nothing itself, so it needs no SMS-sending permission.
- **Phone numbers** stay in the space's encrypted database. They are never uploaded and are left out of anything shared for training.
- **Settings for the reminder:** the business name (the space's name) and an optional till or Paybill number, set once in the space's settings.

### Wording

- Reminder: "Hi {name}, a friendly reminder from {business}: KES {amount} for {what}, due {when}. You can pay to till {till}. Thank you!" Without a till number, the last sentence is left out. Without "what for", it reads "KES {amount}, due {when}".
- The tone is warm and brief, with no pressure words.

## 2. Daily sales and cash

### What people see

- **Business Home** leads with **Sales today** and a split: M-Pesa, Till, Pochi, Cash. Profit this month moves below it.
- **Three quick actions:** "+ Cash", "+ On credit", "Close the day".
- **Add cash sheet:** Cash in or Cash out, a large keypad and a category (Sales or Other income for cash in; the business's cost categories for cash out). One tap saves.
- **Close the day screen:** cash at the start (the float), cash sales, cash spent, and what should be in the drawer. The owner types what they counted. If it differs, the screen shows the amount to account for and offers to add a missed cash spend or keep a note. Below that, M-Pesa and till for the day and total sales.

### How it works

- **A cash account** is added to each business space when it is made (account type `cash` already exists). Cash entries are ordinary transactions on it, so they count in profit, categories and backups like everything else.
- **Sales today** sums income in the business's sales categories since local midnight, grouped by account and, for M-Pesa, by till or Pochi once pesaclarity/pesaclarity-app#149 lands. Until then, till and Pochi payments show under M-Pesa.
- **Day closes** are stored per space: date, float, expected cash, counted cash, difference and note. The next day's float defaults to the counted cash. A day can be closed once and reopened the same day.
- **The difference** is never booked as income or a cost on its own. The owner chooses to add a missed entry or keep the note.

## Out of scope

- Stock and inventory, invoices, several users on one business, and tax. See the business ideas list for later work.
- Sending reminders from our server, and automatic reminders. The owner taps send every time.

## Build order

1. Money owed: credit sales, Money owed screen, payments and part payments, You owe.
2. Reminders and the business name and till settings.
3. Matching incoming M-Pesa payments to customers.
4. Cash account, Add cash sheet and Sales today on Home.
5. Close the day.

Each step ships with tests and screenshots, and the docs and README are updated as it lands.

## Decisions needed

1. Approve the mockups and the reminder wording.
2. Should "Close the day" also send a short daily summary to the owner (a local notification at a set time)? Proposed: not now.
