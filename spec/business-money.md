# Business: mini-POS, records, stock and money owed

**Status:** Proposed · **Date:** 2026-11-05 · **Mockups:** "Business: mini-POS, records, stock and insights" on the app screens canvas · **Related:** [Spaces](income-for-everyone.md#5-spaces-personal-and-business-stage-3), pesaclarity/pesaclarity-app#149

## Why

A business space today shows profit this month, sales and costs by category, and what the owner paid themselves. A small business owner needs more every day:

- what was sold today, and how people paid (cash, M-Pesa, till, Pochi, bank, credit);
- who owes them and whom they owe;
- whether the cash in the drawer is right;
- proper books they can show a Sacco, a bank or an accountant.

The answer is real bookkeeping underneath, with plain, simple screens on top. The owner never sees the words debit, credit or ledger.

## Principles

- **Simple on top, correct underneath.** Every record posts a balanced double entry, so the books always agree.
- **Messages do most of the work.** M-Pesa, till, Pochi and bank payments come in from messages. The owner only adds cash and credit by hand.
- **Ask, don't guess.** When the app can't tell whether money is business or personal, or which customer paid, it asks once and learns.
- **Offline and private.** Everything stays on the phone, in the business space's encrypted database. No new permissions, no server and no privacy policy change.

## 1. Keeping business and personal money apart

Each space has its own database, so business records never show in Personal. Money gets to the right space in three ways, in this order:

1. **By account.** Each account (a till, a bank account, an M-Pesa line) is set to a space: a business, Personal, or "Ask me" for a line used for both. Set in **Business or personal**, and offered when a business space is made.
2. **By till or Pochi.** Payments to a business till or Pochi la Biashara go to that business on their own, once pesaclarity/pesaclarity-app#149 lands.
3. **By person, on a shared line.** On an "Ask me" line, the app asks "Grace Wambui sent you KES 650. Was this for the business?" with **Business sale** and **Personal**, and an "Always do this for Grace Wambui" tick. Each answer is remembered per person or payee, so the questions stop over time. A wrong answer can be fixed by moving the transaction (already built).

A transaction lives in one space only. Moving it between spaces moves its ledger entries with it.

## 2. Getting around: the business bar

A business space swaps the personal tabs (Home, Activity, Plan, Goals, Insights) for its own:

**Today · Money · People · Stock · Insights**

- **Today:** sales today, quick buttons (Bought, Got paid, Paid, Close the day), "take it off what he owes?" questions, and what customers owe and what you owe.
- **Money:** the cash book, with Close the day at the top.
- **People:** Customers and Suppliers, the two other ledgers, as a two-way switch at the top.
- **Stock:** what you sell, products and services.
- **Insights:** how the business is doing, with Full records at the top.
- **Sell:** the round "+" button on Today, Money and Stock opens the mini-POS. It is the same button, in the same place, as "+" in Personal.

**Same design everywhere.** The bar is the personal bar: the same floating pill, icons, label size, colours and shadow. Only the tab names and icons change. Cards, buttons, chips, sheets and type are the shared ones too. Profile and the space switcher stay where they are. Other kinds of space keep the personal bar for now.

## 3. The three ledgers, in plain words

The three ledgers live in Money and People:

| Tab | Ledger | What it shows |
| --- | --- | --- |
| **Money** | Cash book (Cash, M-Pesa and Bank columns) | Balance chips for Cash, M-Pesa and Bank, then the day's entries with how each was paid, and the day's totals in and out |
| **People: Customers** | Sales (debtors) ledger | What customers owe in total and how much is over two weeks old; one row per customer with Remind |
| **People: Suppliers** | Purchases (creditors) ledger | What the business owes, what is due this week, and what was paid this month |

Tapping a customer or supplier opens their **statement**: what they owe, their history (credit sales and payments), their phone and how to remind them.

**Full records** (from the top of Insights) shows the formal views: profit and loss, cash book, customer and supplier statements, and a balance summary. It can be shared as a PDF or exported as CSV.

## 4. Selling and recording

### Sell (mini-POS)

- **Sell** shows what you sell as tiles: name, price, what is left, and "Running low" when below the warning level. Search and category chips sit on top. Tapping a tile adds one, and the count shows on the tile.
- **Other sale** takes a typed amount for anything not in the list, so no one has to pick items at a busy moment. With stock tracking off, Sell is just this amount pad.
- The total sits in a bar where the tab bar was: "4 items · KES 180 · Pay".
- **Pay** lists the items with − and + (part units such as 0.5 kg allowed), then how they are paying: Cash, M-Pesa, Bank, On credit or Split (part now, part later or two ways).
  - **Cash:** an optional "cash given" shows the change.
  - **M-Pesa, till or bank:** the sale waits for the payment message and matches it on its own when the amount and time agree, or the owner taps Done.
  - **On credit:** asks for the customer and when they will pay.

### Other records

The quick buttons on Today:

- **Bought:** stock (adds to stock; see section 5) or a cost, paid now (Cash, M-Pesa, Bank) or "pay later" to a supplier.
- **Got paid:** a customer pays what they owe, fully or in part.
- **Paid:** you pay a supplier, fully or in part.

Sales and purchases from messages are added on their own. When a payment comes from a customer who owes money, Home asks "Otieno paid 1,200 by M-Pesa. Take it off what he owes?". Nothing is applied without a tap.

### What each record posts

| Record | Entries |
| --- | --- |
| Cash sale | Cash up, Sales up; for products, Stock down and cost of goods sold up |
| M-Pesa, till, Pochi or bank sale | That account up, Sales up |
| Credit sale | Customer owes up, Sales up |
| Customer pays | Account up, customer owes down |
| Bought stock | Stock up, account (or owed to supplier) down/up |
| Stock count difference | Stock down, spoiled/used/lost cost up |
| Cost paid now | Account down, that cost up |
| Bought on credit | That cost up, owed to supplier up |
| Pay a supplier | Account down, owed to supplier down |
| Paid yourself | Account down, owner's pay up (counts as income in Personal) |
| Close the day difference (if the owner chooses) | Cash down or up, "cash difference" |

### How it is stored

- A **journal** of entries, each with two or more lines that add up to zero, in each space's own database.
- The existing transactions stay the source: each one posts its entries when it is categorised, and re-posts if it is edited, moved or deleted. Existing business records are posted once when the feature arrives, so nothing has to be re-entered.
- Customers and suppliers build on the existing `LentLoans`, `LoanPayments` and `Debts` tables, with a phone and a reminder choice added.
- A check runs in tests and on the phone: every entry balances, and the three tabs add up to the same totals as the journal.

## 5. Stock: products and services

### One list, two kinds

**What you sell** holds both:

| | Product | Service |
| --- | --- | --- |
| Examples | Tomatoes, sukuma, soap, airtime | Delivery, haircut, repair, tailoring |
| Kept | Stock on hand, unit (piece, kg, bunch, litre), average cost, selling price, "warn me below" | Selling price, or "price varies" |
| Buying | Restock adds to stock at what you paid | Supplies are a cost straight away |
| Selling | Stock goes down; profit = price − average cost | Profit = price; supplies and wages count as costs for the month |
| Warnings | "Running low", days left at the usual pace | None |

Stock tracking is off until the first item is added, so a business that only wants "Sold 350" never sees it.

### How the money works

- **Average cost.** Each product's cost is the average of what was paid, recalculated at each restock: (stock × old cost + new quantity × price paid) ÷ new stock. The Bought sheet shows the new average.
- **Stock is not a cost when bought.** It is something the business owns until it sells. When it sells, its average cost moves from Stock to cost of goods sold, so profit is right even when stock is bought once a month.
- **Part units.** Quantities keep three decimal places, so 0.5 kg or 0.25 litre work.
- **Services** have no stock. Supplies bought for them are a cost when paid.
- **Mixed businesses** (a salon that sells hair products) keep both kinds in one list. Reports split product profit and service profit.

### Screens

- **Stock tab:** stock value and how many items are running low, an All / Products / Services filter, one row per item, and Add an item. Count stock sits at the top.
- **Add an item:** Product or Service, name, price and unit, about what you pay (products), stock now and "warn me below".
- **One item:** stock now, days left, Restock and Count; selling price, average cost, profit per unit, sold this week; history of sales, restocks and counts.
- **Bought (stock):** item, how much, what you paid, paid with (Cash, M-Pesa, Bank, Pay later), and the new stock and average cost.
- **Count stock:** what the records say against what you count. A difference is kept as Spoiled, Used, Lost or Miscounted and, except Miscounted, counts as a cost at the average cost.

### Out of scope for stock

Barcodes, several shops or stores, batches and expiry dates, and buying in one unit and selling in another (a sack sold by the kg). These can come later.

## 6. Start your records (opening balances)

When a business space is made, and for existing business spaces the first time Records opens, **Start your records** asks once for:

- cash in the drawer;
- the business M-Pesa or till balance (filled in from the latest message when possible);
- the business bank balance;
- customers who already owe the business and suppliers it owes (now or later);
- stock on hand, for businesses that track items (now or later).

These post as opening entries. They can be changed later, and changes are kept in the history.

## 7. Reminders

- **Message templates** per business. Three to start ("Friendly", "Due today", "It's been a while"), each editable, and the owner can add more. Fields fill in on their own: {name}, {business}, {amount}, {what}, {due}, {date}, {till}. A sentence whose field is empty is left out.
- **Send with:** WhatsApp or SMS, chosen per customer, with a default for the business. The Remind sheet lets the owner pick the template and the app, and change the words for that one message.
- **Daily nudge:** at a time the owner sets, a local notification lists who is due, with "Remind all who are due" stepping through them one by one.
- **The owner always taps send.** The app opens WhatsApp (`wa.me` link) or the SMS app (`sms:` intent) with the message filled in. It never sends by itself, so it needs no SMS-sending permission, no WhatsApp Business account and no server.
- Starting wording:
  - Friendly: "Hi {name}, a friendly reminder from {business}: KES {amount} for {what}, due {due}. You can pay to till {till}. Thank you!"
  - Due today: "Hi {name}, KES {amount} is due today. Pay to till {till} when you can. Thank you!"
  - It's been a while: "Hi {name}, just checking in on KES {amount} from {date}. Pay to till {till} when you can. Thank you!"

## 8. Close the day

Cash at the start (yesterday's counted cash, or the opening balance), plus cash in, minus cash out, gives what should be in the drawer. The owner types what they counted. If it differs, the app shows the amount to account for and offers to add a missed entry or keep a note. It never books the difference on its own. M-Pesa and till for the day and total sales show underneath. Day closes are kept per space.

## 9. Insights

On Home and in **How the business is doing** (this month, last month, 3 months):

- **Sales today** with the split by how people paid, compared with a usual day of that weekday.
- **Profit** for the period and the change on the last one.
- **How people pay:** share of sales by cash, M-Pesa, till, Pochi, bank and credit.
- **Best sellers:** top items by sales, with quantity and profit per unit; services marked.
- **Sales by day of the week,** with the best day marked.
- **Short sentences**, each from the journal:
  - best day and when cash sales peak;
  - product profit against service profit, and items that sell slowly;
  - stock value, and items that will run out before the usual restock day;
  - cash in and out for the period;
  - customers owing for more than two weeks, and who usually pays on time;
  - supplier payments due, and whether the expected M-Pesa balance covers them;
  - how often the drawer matched.
- **Share a report:** a one-page PDF with profit and loss, money owed and the balance summary, for a Sacco or a loan.

## Out of scope

Invoices, VAT and tax, several users or tills on one business, receipt printers, and sending messages from our server. For stock, see section 5.

## Build order

1. Journal and posting underneath, with existing business records posted and balance checks. No visible change.
2. Business or personal: account settings and the question on shared lines.
3. Start your records (opening balances).
4. The business bar, Sell with "Other sale" and Pay (no items yet), Bought, Got paid and Paid, and the Money tab.
5. People: Customers and Suppliers, with statements and matching incoming payments.
6. Stock: items, Sell tiles, average cost, restock and count.
7. Reminders: templates, send with, daily nudge.
8. Close the day.
9. Insights, Full records, PDF and CSV.

Each step ships with tests, screenshots, and docs and README updates.

## Decisions needed

1. Approve the mockups and the three starting reminder messages.
2. Split payment on Pay (part now and part on credit, or two ways such as cash and M-Pesa). Proposed: yes.

Agreed so far: bank sales tagged from messages with an "Is this a sale?" check; opening balances; average cost; part units such as 0.5 kg; the business bar matching the personal one.
