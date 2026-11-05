# Business: mini-POS, records, stock and money owed

**Status:** Accepted · **Date:** 2026-11-05 · **Mockups:** "Business: mini-POS, records, stock and insights" and "Services: hours, attached documents and tax" and "Services: hours day by day" on the app screens canvas · **Related:** [Spaces](income-for-everyone.md#5-spaces-personal-and-business-stage-3), pesaclarity/pesaclarity-app#149

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

## 2. What the business sells

When a business space is made, **What do you sell?** asks once: **Products**, **Services** or **Both**. It sets which screens appear, and can be changed later in the space's settings.

| | Products | Services | Both |
| --- | --- | --- | --- |
| Bar | Today · Money · People · Stock · Insights | Today · Money · Clients · Services · Insights | Today · Money · People · Stock · Insights |
| Stock, counts, restocking | Yes | No | For products only |
| Today's buttons | Bought, Got paid, Paid, Close the day | Got paid, Spent, Billed, Pay yourself | As Products |
| "+" button | Sell (mini-POS) | Got paid | Sell, with services as tiles |
| Profit | Sales − cost of goods sold − other costs | Income − costs | Product profit and service profit, shown apart |

**Services businesses** (a consultant, a salon, a mechanic's labour, transport):

- **Services** tab: what you offer and your rate (per session, day, month or "price varies"), and what each earned this month.
- **Clients** tab: the sales ledger under a fitting name, with statements and reminders as for customers. Suppliers sit inside it as a second switch for those who use them.
- **Billed:** work done that a client will pay later (it becomes what they owe). No invoice document for now.
- **Spent:** a business cost, paid now or later.
- Close the day is hidden unless the business takes cash.

## 3. Pay yourself, and what Personal sees

- **Pay yourself** moves money from the business to the owner: amount, from which business account, and how it shows in Personal ("Salary from my business" or "Other income"). The sheet shows profit this month and what stays in the business.
- In the business it posts as owner's pay: money leaves, but it is not a business cost, so profit stays right.
- In **Personal** it arrives as income, tagged with the business's name. Personal then uses it like any income: Home shows "From Amina Consulting", the plan and "until the next money" use the usual monthly amount, and insights compare business pay with spending ("At this pace your business pay covers 1.4 months of your spending").
- Personal sees only what was paid out, never the business's own records. When the business account is also the owner's M-Pesa line, the transfer is still one record: a "Pay yourself" entry in the business and the matching income in Personal, linked so neither is counted twice.

## 4. Getting around: the business bar

A business space swaps the personal tabs (Home, Activity, Plan, Goals, Insights) for its own:

**Today · Money · People · Stock · Insights**

- **Today:** sales today, quick buttons (Bought, Got paid, Paid, Close the day), "take it off what he owes?" questions, and what customers owe and what you owe.
- **Money:** the cash book, with Close the day at the top.
- **People:** Customers and Suppliers, the two other ledgers, as a two-way switch at the top.
- **Stock:** what you sell, products and services.
- **Insights:** how the business is doing, with Full records at the top.
- **Sell:** the round "+" button on Today, Money and Stock opens the mini-POS. It is the same button, in the same place, as "+" in Personal.

**Same design everywhere.** The bar is the personal bar: the same floating pill, icons, label size, colours and shadow. Only the tab names and icons change. Cards, buttons, chips, sheets and type are the shared ones too. Profile and the space switcher stay where they are. Other kinds of space keep the personal bar for now. Tabs join the bar as they are built: Today, Money and Insights first, then People (step 5), then Stock or Services (step 6).

## 5. The three ledgers, in plain words

The three ledgers live in Money and People:

| Tab | Ledger | What it shows |
| --- | --- | --- |
| **Money** | Cash book (Cash, M-Pesa and Bank columns) | Balance chips for Cash, M-Pesa and Bank, then the day's entries with how each was paid, and the day's totals in and out |
| **People: Customers** | Sales (debtors) ledger | What customers owe in total and how much is over two weeks old; one row per customer with Remind |
| **People: Suppliers** | Purchases (creditors) ledger | What the business owes, what is due this week, and what was paid this month |

Tapping a customer or supplier opens their **statement**: what they owe, their history (credit sales and payments), their phone and how to remind them.

**Full records** (from the top of Insights) shows the formal views: profit and loss, cash book, customer and supplier statements, and a balance summary. It can be shared as a PDF or exported as CSV.

## 6. Selling and recording

### Sell (mini-POS)

- **Sell** shows what you sell as tiles: name, price, what is left, and "Running low" when below the warning level. Search and category chips sit on top. Tapping a tile adds one, and the count shows on the tile.
- **Other sale** takes a typed amount for anything not in the list, so no one has to pick items at a busy moment. With stock tracking off, Sell is just this amount pad.
- The total sits in a bar where the tab bar was: "4 items · KES 180 · Pay".
- **Pay** lists the items with − and + (part units such as 0.5 kg allowed), then how they are paying: Cash, M-Pesa, Bank, On credit or Split (part now, part later or two ways).
  - **Cash:** an optional "cash given" shows the change.
  - **M-Pesa, till or bank:** the sale waits for the payment message and matches it on its own when the amount and time agree, or the owner taps Done.
  - **On credit:** asks for the customer and when they will pay.

### When the payment doesn't match the sale

Pay compares what came in with the sale and asks only when they differ:

| What happened | The choice | What it posts |
| --- | --- | --- |
| Paid more (sale 180, M-Pesa 200) | **Gave change in cash**, **Gave change by M-Pesa**, **Keep it as a tip**, **I owe it back** | Change: that account down 20; tip: other income 20; owe back: owed to the customer 20. The sale stays 180 |
| Paid less (sale 500, M-Pesa 450) | **They owe it**, **Paid the rest in cash**, **Another payment**, **Gave a discount** | Customer owes 50; or cash 50; or waits for a second payment; or discount 50 |
| Paid in two or more messages | **Choose payments:** recent unmatched M-Pesa, till and bank payments with a tick each and a running total ("100 + 80 = 180") | All the payments linked to the one sale |
| One payment for several sales or old debts | Tick what it covers; anything left becomes change, credit or other income | Split across them |
| Matched to the wrong sale | Open the sale, **Unlink payment**, choose again | Entries re-posted |

- Only payments from the last few hours that are not yet matched are offered, newest first, so the list stays short.
- **Close the day** lists payments not matched to any sale, each with **Match** (to a sale, to a customer who owes, or kept as other income), so nothing is left loose.

### Other records

The quick buttons on Today:

- **Bought:** stock (adds to stock; see section 7) or a cost, paid now (Cash, M-Pesa, Bank) or "pay later" to a supplier.
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

## 7. Stock: products and services

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

## 8. Start your records (opening balances)

When a business space is made, and for existing business spaces the first time Records opens, **Start your records** asks once for:

- cash in the drawer;
- the business M-Pesa or till balance (filled in from the latest message when possible);
- the business bank balance;
- customers who already owe the business and suppliers it owes (now or later);
- stock on hand, for businesses that track items (now or later).

These post as opening entries. They can be changed later, and changes are kept in the history.

## 9. Reminders

- **Message templates** per business. Three to start ("Friendly", "Due today", "It's been a while"), each editable, and the owner can add more. Fields fill in on their own: {name}, {business}, {amount}, {what}, {due}, {date}, {till}. A sentence whose field is empty is left out.
- **Send with:** WhatsApp or SMS, chosen per customer, with a default for the business. The Remind sheet lets the owner pick the template and the app, and change the words for that one message.
- **Daily nudge:** at a time the owner sets, a local notification lists who is due, with "Remind all who are due" stepping through them one by one.
- **The owner always taps send.** The app opens WhatsApp (`wa.me` link) or the SMS app (`sms:` intent) with the message filled in, to the customer's number with its country code (see **Sending to a number** in section 14). It never sends by itself, so it needs no SMS-sending permission, no WhatsApp Business account and no server.
- Starting wording:
  - Friendly: "Hi {name}, a friendly reminder from {business}: KES {amount} for {what}, due {due}. You can pay to till {till}. Thank you!"
  - Due today: "Hi {name}, KES {amount} is due today. Pay to till {till} when you can. Thank you!"
  - It's been a while: "Hi {name}, just checking in on KES {amount} from {date}. Pay to till {till} when you can. Thank you!"

## 10. Close the day

Cash at the start (yesterday's counted cash, or the opening balance), plus cash in, minus cash out, gives what should be in the drawer. The owner types what they counted. If it differs, the app shows the amount to account for and offers to add a missed entry or keep a note. It never books the difference on its own. M-Pesa and till for the day and total sales show underneath. Day closes are kept per space.

## 11. Insights

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

## 12. Make a report

Reports keep one clear, consistent look in the app's style. The owner chooses what goes in, not the layout:

- **Who is it for:** "A Sacco or loan" (profit and loss, money owed, balance summary), "An accountant" (everything, including every cash book entry) or "Just me" (insights and charts). Each ticks sensible sections, which the owner can change.
- **Period:** this month, last month, 3 months, or dates.
- **Sections**, each a tick: profit and loss, sales by how people paid, customers who owe, suppliers owed, balance summary, best sellers, stock, cash book with every entry, insights.
- **The cover:** business name, owner name, phone, place and an optional logo or photo.
- **Detail:** totals only, or every entry.
- **Format:** PDF to share, or CSV for a spreadsheet.
- **Saved setups,** such as "Monthly Sacco report", to make the same report again in one tap.

Free-form layouts and colours are out of scope, so every report looks professional.

## 13. Tax estimates (Kenya, for SMEs)

Approximations to plan with, never official records or advice. Every figure is marked **Estimate**, with "Based on your records and the rules as of {date}. Check with KRA or a tax adviser."

**Version one:**

- **Which tax applies:** from the last 12 months of sales, the likely band: below the Turnover Tax (TOT) threshold, TOT, or above it (income tax on profit, later). A warning before a line is crossed, such as nearing the VAT registration threshold.
- **TOT this month:** the rate times gross sales for the month, with the due date and a reminder a few days before.
- **Set aside for tax:** an optional pot that keeps a share of each sale aside.
- **Withholding tax for services:** see **Services: fees, withholding tax and income tax** below.
- **Payments to KRA** recognised from M-Pesa and bank messages and marked as tax paid.
- **Estimated against paid,** month by month, and how many costs have eTIMS invoices.
- **Tax summary** as a section in Make a report.

### Services: fees, withholding tax and income tax

For consultants and other businesses paid fees for their time or skill. Turnover Tax does not cover professional and consultancy fees, so a services business gets an income tax estimate instead. The rules below are as currently understood and live in the tax pack, like every rate.

- **You file as:** Individual (individual rates and personal relief) or Company (the company rate), in Tax settings. It changes only which rates the estimate uses.
- **Withholding tax, per invoice,** remembered per client:
  - **No withholding tax:** they pay the full invoice.
  - **Client deducts it:** they pay the fees less the rate (5% for resident consultancy fees today) and pay that share to KRA. "To pay" shows the lower figure.
  - **Client pays it on top:** the fees are grossed up, fees ÷ (1 − rate), so after they deduct it the owner still gets the agreed fees. 110,000 shows as 115,790 and 5,790 is withheld.
  - Withholding applies to fees only; costs passed on are not included.
- **When the payment comes:**
  - A payment equal to "To pay" settles the invoice, and the withheld share is recorded as tax already paid.
  - A payment short by exactly the rate's share of the fees, on an invoice with no withholding set, opens **Paid less** with **Tax withheld (5%)** first, and "Expect 5% from {client} from now on".
  - Each withheld amount can carry its **certificate number**. The Tax screen lists certificates still to collect, by client.
- **The Tax screen:**
  - fees earned, costs and profit for the year to date;
  - income tax on profit, as an estimate;
  - less withheld by clients, and less paid to KRA (recognised from payment messages);
  - what is left to pay, with the next instalment date from the tax pack.
- **VAT:** "VAT registered" in Tax settings adds a 16% VAT line on invoices (the rate from the tax pack). VAT returns stay out of scope.

Later: income tax bands for product businesses above the TOT band, and VAT estimates for registered businesses.

**The tax pack.** Rates, thresholds and due dates live in a dated, versioned tax pack inside the app, each figure with its source. Nothing is hardcoded in code, and the pack never comes from our server: it is reviewed before each release and updated only with app updates. The app shows "Rules as of {date}" wherever an estimate appears.

## 14. Receipts and invoices

Made on the phone, never online. Nothing goes to our server: the app builds the document and opens the phone's share menu, so the owner sends it by WhatsApp, SMS, email, Bluetooth, a printer app or any app they choose.

### Receipts

- **When:** after Pay ("Sale saved", with **Share receipt** and **Not now**), and later from any sale. "Offer a receipt after each sale" can be switched off.
- **What it shows:**
  - logo, business name, place, phone, and till or Paybill;
  - receipt number, date and time;
  - items as quantity × price, and the total;
  - how they paid, with the M-Pesa code from the matched message;
  - change given, or the balance still owed;
  - the owner's footer line, and "This is not an eTIMS tax invoice".
- **As:** a narrow picture sized for WhatsApp, a PDF, or plain text for SMS. The customer's number is filled in when the sale has a known customer.

### Invoices

- **Make an invoice** from Billed (services), a credit sale, or Invoices (on Clients or People):
  - the client and, optionally, the project;
  - **bill by logged hours or by items.** Logged hours come from **Hours** (below): the hours not billed yet for the client, from and to a date, ticked by project. By items, quantity × rate;
  - **costs passed on** to the client, such as a software subscription bought for the work, each with its receipt attached;
  - subtotal and any discount;
  - withholding tax: none, deducted by the client, or paid by the client on top (section 13), giving "To pay";
  - due date, a number in order, and how to pay (Paybill or till with the invoice number as the account, and bank details);
  - the business's **M-PESA QR code** next to how to pay, labelled "Scan with the M-PESA app";
  - notes.
- **Attached documents:** PDFs or pictures from the phone's files (for example a receipt saved from email, or the eTIMS invoice made on KRA's site) or a photo of a paper receipt.
  - Attaching a receipt can add it as a cost line ("Charge it to the client") and record it as a business cost if Money doesn't have it yet.
  - Attachments are kept encrypted with the business space, included in its backup, and never uploaded.
- **Save or share:**
  - **One PDF:** the invoice, then the timesheet page when billed by hours, then the attachments in order. Or **separate files**.
- **Show hours on the invoice** as **Total only** (one line per project), **By project** (each project's hours and rate), or **Day by day** (each day with what was done). A preview shows the choice. **Add the full timesheet as a page** puts every entry in the PDF, with day totals, project totals and the grand total, so the client sees the breakdown even when the invoice shows one line.
  - **Save to phone** puts it in Files or Downloads through the phone's own save screen. **Share** opens the share menu.
- **Status:** Unpaid, Part paid, Paid or Overdue. It changes on its own when a payment message matches.
- **The Invoices list** shows what is waiting and what is overdue.
- **Reminders** can attach the invoice.
- **Paid invoices** can be shared again with a Paid stamp.
- **Customer statement:** a PDF of a customer's invoices, payments and balance, from their People screen.

### Hours (services)

For work billed by the hour. Logged on the phone, kept with the business.

- **Log hours:** client, project, what was done (a meeting, a task), the day, and hours (halves and quarters allowed). "Bill this to the client" is ticked by default, at the rate saved for that client and project; travel or admin can be logged unbilled. "Log another for the same day" keeps the day for the next entry.
- **A day:** every entry for the day, across projects and meetings, with the day total and what is to bill per client.
- **A week:** each day's total with its split by client, then totals **by project** (hours × rate and amount) and the unbilled hours, with **Bill these hours**.
- **Today** (services) shows **Log hours** beside Got paid, Spent and Billed, and a card with hours this week and hours not billed yet.
- **Billing:** an invoice takes the logged hours it covers, and they are marked billed with that invoice's number. Cancelling the invoice frees them. Hours can be edited until billed.
- Hours live under the **Services** tab, with the services and rates.

### Sending to a number

Receipts, invoices, statements and reminders can go to a phone number:

- **Country code:** a picker beside the number, starting with the phone's own country (+254 in Kenya), searchable by country or code. Numbers are kept in full international form, so WhatsApp links work for clients abroad.
- **Choose from contacts:** a button opens the phone's own contact picker. The app gets only the one contact chosen, keeps its number (with its own country code) on that customer or client, and never reads or uploads the contact list.
- The number is filled in when the sale, invoice or customer already has one.

### Settings (Receipts and invoices)

Logo (initials or a photo from the phone), business details, how to pay, the M-PESA QR code, KRA PIN shown or hidden, numbering prefix, receipt footer, and the default receipt format. Content changes, layout does not, so every document looks professional.

### The M-PESA QR code

Safaricom makes official M-PESA QR codes for a till or Paybill, and its API for QR codes with an amount filled in works online. So, offline only:

- The owner adds **their own QR code** once, from the M-PESA business app or the printout Safaricom gave them: a photo or screenshot, cropped on the phone. The app reads it to check that it is a QR code and shows it on invoices (and optionally receipts) exactly as given.
- The app never makes its own payment QR codes, so a customer only ever scans Safaricom's code. The amount is typed by the customer, and the invoice shows it beside the code.
- Without a QR code added, the How to pay box shows the till, Paybill and account only.

### Look

A4 invoices have a thin accent strip at the top (agreed). Receipts and invoices share the report's type, colours and spacing.

### Rules

- Numbers are in order per business and never reused. A cancelled document keeps its number and is marked Cancelled.
- Documents are business documents, not KRA eTIMS tax invoices, and say so. eTIMS needs KRA's online system, so it is out of scope while documents are offline only. An eTIMS invoice made on KRA's site can be attached.
- **No new permissions.** Files are picked and saved through the phone's own file screens, photos through its camera app, and contacts through its contact picker, none of which need storage, camera or contacts permissions. The permissions test stays as it is.
- **Privacy policy:** before attachments and picked contacts ship, the policy gains a line on customer and client details and attached documents: kept on the phone, encrypted, in backups the owner controls, never sent to us.
- Documents are rebuilt from the records when shared again, so a corrected sale gives a corrected receipt with the same number and "Corrected" on it.

## Out of scope

VAT returns and filing, several users or tills on one business, receipt printers, and sending messages from our server. For stock, see section 7.

## Build order

1. Journal and posting underneath, with existing business records posted and balance checks. No visible change.
2. Business or personal: account settings and the question on shared lines.
3. What do you sell, and Start your records (opening balances).
4. The business bar, Sell with "Other sale" and Pay (no items yet), Bought, Got paid and Paid, and the Money tab.
5. People (Clients for services), statements, Pay yourself into Personal, and payment matching: paid more, paid less, choose payments.
6. Stock: items, Sell tiles, average cost, restock and count.
7. Reminders: templates, send with, daily nudge.
8. Close the day.
9. Insights, Full records, and Make a report (PDF and CSV).
10. Tax estimates and the tax pack, with the services estimate: you file as, withholding per client and invoice, certificates, and the Tax screen.
11. Receipts (picture, PDF, text) and invoices, with statuses, statements and document settings; logged hours by day and by week, billing by hours with total, by project or day-by-day detail and a timesheet page, costs passed on, attached documents, one PDF or separate files, save to phone, and sending to a number with a country code or from contacts.

Each step ships with tests, screenshots, and docs and README updates.

## Decisions needed

1. Approve the mockups and the three starting reminder messages.
2. Split payment on Pay (part now and part on credit, or two ways such as cash and M-Pesa). Proposed: yes.

3. Services businesses: is "Billed" (work done, pay later) enough, or are invoice documents needed now? Proposed: Billed now, invoices later.

Agreed: split payments both ways (part now and part on credit, or two ways such as cash and M-Pesa); Billed for services makes an invoice; invoices with a thin top strip and the M-PESA QR; receipts and invoices made offline and shared through any app; tax version one (TOT, VAT threshold warning, withholding tax for services, KRA payments) with the tax pack shipped in the app only; Products / Services / Both; Pay yourself feeding Personal; bank sales tagged from messages with an "Is this a sale?" check; opening balances; average cost; part units such as 0.5 kg; the business bar matching the personal one; hours logged day by day (per project, or several projects and meetings in a day) and billed with the detail the owner chooses; invoices with costs passed on and attached documents, saved to the phone as one PDF; withholding tax per client and invoice (none, deducted, or paid on top) with an income tax estimate for services; country codes and the contact picker for numbers.
