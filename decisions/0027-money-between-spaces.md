# 0027. Money on one account that belongs to another space

Date: 2026-11-21 · Status: Accepted · Builds on [0016](0016-spaces-kept-apart.md), [0018](0018-double-entry-journal.md) and [0026](0026-mpesa-family-of-accounts.md)

Accepted by the owner on 21 November 2026, with the recommendations and the decisions below, and three rules the owner added: a transaction can be switched to another space and then shows only there (section 2); a business can always pay its owner (section 6); and the owner can spend the business's money directly (section 6). On the same day the owner answered how Personal counts that last case: as spending only (decision 20). Nothing here is built yet. Screens need approved mockups before they are built. The channel rules wait on the owner's own samples.

It changes how 0016 moves a payment: on screen it still leaves the first space, but the first space keeps what its account's balance needs. The business money spec's line that a transaction lives in one space only still holds for everything a person sees, and gains a second: an account lives in one space only.

## Context

A shop owner often takes the shop's money on their own M-Pesa line. Pesa Clarity reads that line in Personal, and the wallet's balance is everything the owner has on it. An audit of the app on 21 November 2026 (`pesaclarity-app` `origin/dev`) found that the app has no way to say "this money is on my account, and it is the shop's". What exists today:

- **Spaces are separate files** (0016). Each has its own accounts, payments and journal. An account is set to one space, or to "Used for both": Personal reads it, and the business asks "Grace Wambui sent you KES 650. Was this for the business?" (`RoutingRules`, `SharedLineRouter`).
- **A payment crosses by being moved.** `DriftSpaceMoveRepository.move` copies the row into the other space and deletes it in the first. The copy keeps the balance its message printed ("New M-PESA balance is Ksh38,469.00").
- **Balances follow reported balances** (0018). `WalletWalk` takes each reported balance as what the account holds at that moment. That is right for the wallet and wrong for a copy in another space.
- **So one sale breaks three figures.** The owner's wallet holds KES 37,819. A customer pays KES 650, and the wallet holds KES 38,469. The owner answers "Business sale". The business now shows M-Pesa KES 38,469, the owner's whole wallet. Personal's M-Pesa drops back to KES 37,819 until its next message, then shows KES 650 its records cannot explain. The two spaces together show KES 76,288 for a wallet that holds KES 38,469.
- **The owner cannot put it right.** Start your records offers the business the wallet's balance. A typed opening balance, or a Set balance correction, lasts until the next payment is answered. A business cost that empties the wallet is read, in the business, as Fuliza covering it: a debt nobody took.
- **The answer is not a sale.** The moved payment arrives with no category, so the business holds it as "not yet sorted" until it is filed a second time.
- **Pay yourself waits for messages.** Cash is recorded on both sides at once. M-Pesa and bank wait for a message in each space. On a shared line no money moves, so no message comes: nothing is recorded, or the next payment of the same amount is filed as the owner's pay. The spec says this should be one linked record. From a business bank into the owner's M-Pesa, one "From" answer serves both sides, so one side never matches.
- **Money the owner puts in has no name.** A business has "Paid yourself" for money out. Money in from the owner counts as business income, which raises profit and the tax estimate, and as spending in Personal.
- **A bank in one space and M-Pesa in the other.** KCB also texts about the M-Pesa side of a transfer. A business that reads only KCB is given an M-Pesa account for a wallet it does not have, and the transfer is paired inside the business: what it sends its owner never leaves it.
- **One line, the business's own channels.** Many owners have no line of their own for the business. Its money still comes in its own way: into the owner's Pochi la Biashara, a Lipa na M-Pesa till (Buy Goods) or a paybill, while money sent to the owner's number and the owner's own payments are personal. The app cannot tell these apart. The `mpesa` pack (0.3.1) reads only the payer's side of a till, a paybill or someone else's Pochi; no rule reads money received through the owner's own Pochi, till or paybill, and the app has no Pochi at all. Such a message reads as ordinary money received, or goes to the review inbox.
- **Giving a line to a business copies its past.** The business reads every message on the phone again. Personal keeps its payments and its last balance. Both then count the wallet.
- **The books have nowhere to put it.** 0018 has Not yet sorted, Money in transit and Paid yourself. It has no account for money one space holds for another. 0016 and the spec say where a payment lives. Neither says whose balance it changes.

The question: when KES 650 for the shop lands on the owner's line, what should Personal and the business each show, in the list of payments and in the books, so that balances, money in and out, profit and safe to spend are right, and the two spaces together never count a shilling twice or lose one?

## Decision

Four ways were worked through with the same numbers. They are compared under Alternatives. The owner chose the first: **the account stays whole in one space, and the other space gets the transaction, on an owner's account.**

The same KES 650 sale under each (the wallet holds KES 38,469 after it):

| | Personal shows | The business shows | Together |
| --- | --- | --- | --- |
| Today | M-Pesa 37,819 until its next message, then the whole wallet with 650 unexplained | M-Pesa 38,469; the sale not yet sorted | 76,288 |
| **A. One home, linked entry (chosen)** | M-Pesa 38,469, of which the business's 650; money in 0 | On the owner's M-Pesa 650; Sales 650 | 38,469 |
| B. Shares of the line | Its share of M-Pesa 37,819; money in 0 | Its share of M-Pesa 650; Sales 650 | 38,469, known only by adding two spaces |
| C. Paid to the owner at once | M-Pesa 38,469; money in 650, as pay | Sales 650; Paid yourself 650; holds nothing | 38,469; the business's part is unknown |
| D. Move, without the balance | As today | M-Pesa 650; Sales 650 | Right until Personal's next message, then the 650 counts twice |

What A costs: two new accounts and one new role in the books, one linked write, a change to every screen that adds up money, and a repair of payments already moved. No balance is ever checked across two spaces.

### 1. An account lives in one space

Every account (an M-Pesa line, a bank account, cash) lives in exactly one space. That space alone reads its messages and holds its balance. A transaction on it can still be switched to any space, and then shows only there (section 2). Every reported balance is checked there, against all of the account's payments (0018). No other space has an account for it.

"Used for both" keeps its meaning for the person and gets a precise one for the app: the line lives in Personal, and some of what is on it belongs to the business.

A channel with a balance of its own, such as the owner's Pochi la Biashara (0026), is an account of its own and lives in the business. Money that lands in the wallet itself has no account of its own; that is what the rest of this record is about. Section 4 covers both on one line.

### 2. A payment that belongs to another space

**One transaction, one space.** A person can switch any transaction to another space. From then on it shows only in that space: in its Activity, its totals, its reports, its search and its exports. No transaction is ever a row in two spaces. Answering "Business sale" to "Was this for the business?" is the same switch, made when the payment arrives.

When the payment was read on an account that lives in another space, the switch records two things, with one shared link:

- **In the space it was switched to**, the transaction itself: the sale or the cost, with its amount, time, the other person's name, its category, the fee and the M-Pesa code. It carries no balance and sits on no wallet. It sits on the owner's account (below).
- **Where the account lives**, a balance record, not a transaction. It keeps only what the account's balance needs: the amount, the time, the fee, the M-Pesa code, the balance the message printed, the link, and the space the payment went to. It keeps no name, no category and no note. It is kept in that space's file, on the account (here Personal's M-Pesa), and posts one entry to that space's journal: for a sale, M-Pesa up and Held for the stall up; for a cost, both down. So the account's balance still adds up, each reported balance is still checked, and the message is never read in again as a new payment. It is in no list, total, report, search or export. On screen it shows only inside M-Pesa's balance and in the line "KES 650 of this is Vegetable stall's".

**Switching back.** "It was mine after all", or switching the sale to Personal, turns the balance record back into a transaction in Personal, with the details the business's entry held. The business's entry is removed. Both journals post a reversal (0018), and Held for the stall is put back as it was before the switch. A transaction can be switched as often as the person likes; it is always in exactly one space.

A transaction typed by hand carries no reported balance, so it moves outright and leaves nothing behind. A payment out of the business's own account that was for the owner is the one exception, with an entry in each space, each meaning something different (section 6).

Each side needs one new account in its books:

| Space | Account | What it holds |
| --- | --- | --- |
| Personal | **Held for Vegetable stall** (one for each other space) | Money on Personal's accounts that is the stall's. Below zero, it is what the stall owes the owner |
| The business | **Owner's account** | The stall's money that the owner holds. Below zero, it is what the owner has put in |

The two always show the same amount. That is the rule that replaces a reported balance for the business's side: the app checks it whenever both spaces can be opened.

What each event posts ("up" and "down" as in the business money spec):

| Event | Where the account lives (Personal) | The other space (the business) |
| --- | --- | --- |
| Money in for the business | M-Pesa up, Held for the stall up | Owner's account up, Sales (or the category chosen) up |
| A business cost paid from the line | M-Pesa down, Held for the stall down | That cost up, Owner's account down |
| A fee on such a payment | Follows the payment | Fees up, Owner's account down |

Nothing posts across spaces: each entry balances inside its own journal, as 0018 requires. What is new is that two journals each hold an entry about one event. Only one of them is a transaction anyone sees.

A personal payment from an account that lives in the business, and a business cost paid from the owner's line, are section 6.

### 3. The worked example

Made-up numbers. The owner's M-Pesa lives in Personal and holds KES 37,819, all the owner's. The stall has KES 2,000 in its cash drawer and KES 10,000 in its KCB account.

| Step | The wallet says | Personal | Vegetable stall | Check |
| --- | --- | --- | --- | --- |
| 1. Grace Wambui pays KES 650 for goods | 38,469 | M-Pesa 38,469. Held for the stall 650. Money in 0 | Owner's account 650. Sales 650. Profit 650 | 37,819 the owner's + 650 the stall's |
| 2. The owner pays a supplier KES 400 from the line | 38,069 | M-Pesa 38,069. Held for the stall 250. Money out 0 | Stock and supplies 400. Owner's account 250. Profit 250 | 37,819 + 250 |
| 3. Pay yourself KES 200, on the same line | 38,069 | Held for the stall 50. Salary from my business 200 | Paid yourself 200. Owner's account 50. Profit still 250 | 38,019 + 50 |
| 4. Pay yourself KES 2,500, from the stall's KCB into the owner's M-Pesa | 40,569 | M-Pesa 40,569. Salary from my business 2,500. Held for the stall 50 | KCB 7,500. Paid yourself 2,500. Owner's account 50 | 40,519 + 50 |

Had the cost in step 2 been KES 1,000, the stall would have spent more than it had on the line. Held for the stall would be 350 below zero. Personal says "Vegetable stall owes you KES 350". The stall says "You have put in KES 350". The wallet holds KES 37,469, all of it the owner's.

In steps 1 and 2 Personal's Activity shows nothing new: the sale and the cost are rows in the stall only, and Personal keeps their balance records. Steps 3 and 4 are Pay yourself, a transfer between spaces: each space shows its own side (section 6).

In step 3 no money moves and no message comes, so both sides are recorded at once, as cash is today. In step 4 real money moves: KCB's message is read in the stall and M-Pesa's in Personal. The stall is never given an M-Pesa account for it.

The next three start again from step 3: the wallet holds KES 38,069, of which KES 50 is the stall's.

**5. Set balance on a shared line.** There are now two different corrections.

- *The wallet is off.* M-Pesa says KES 37,869 and the app shows KES 38,069. Set balance in Personal records a balance correction of KES 200 in Personal. The stall's part stays KES 50. If the missing payment turns up and was the stall's, it is added in the stall, Personal keeps its balance record, and the correction is undone.
- *The stall's part is off.* The owner knows KES 300 of the wallet is the stall's, not KES 50. "Set the stall's part" records a linked pair: Owner's account up 250 in the stall, Held for the stall up 250 in Personal, each against Balance corrections. The wallet is not touched: KES 38,069 is KES 37,769 the owner's and KES 300 the stall's.

The business can no longer set an M-Pesa balance for this line, because it has no M-Pesa account for it.

**6. Switching an old payment.** On 3 September the owner paid Example Suppliers KES 4,000 from the line, and Personal counted it as spending. Today the owner switches it to the stall. In Personal it leaves September's Activity, search and exports, and September's spending goes down by KES 4,000. Its balance record stays on M-Pesa in September, so September's balances still add up. The stall gets a cost of KES 4,000 dated 3 September, so its September profit goes down by KES 4,000. Held for the stall is now 3,950 below zero: the stall owes the owner KES 3,950. Both journals post a reversal and a new entry (0018). A month already frozen in the Plan stays as it was.

**7. The line is later given wholly to the business.** On 1 December the owner sets M-Pesa to "Only Vegetable stall". The hand-over is one dated event:

- Personal: M-Pesa closes at KES 38,069 and shows nothing from then on. Its history stays. Held for the stall goes down by 38,069, to 38,019 below zero.
- The stall: M-Pesa opens on 1 December at KES 38,069, so its first message's balance agrees. Owner's account goes down by 38,069: "You have put in KES 38,019".
- Nothing from before 1 December is read into the stall. It already has its own sales and costs from that time.

The owner's KES 38,019 is now money they have put into the stall. They can leave it there or send it to another account of their own, which brings both sides back to zero. The app says so before the hand-over and suggests moving it out first.

### 4. One line, with the business's own channels

This is the usual case, not an edge: one M-Pesa line holds the owner's money and the business's, and the business's money comes in through its own channel.

**The channel decides first.** Within a line:

| Money in through | Goes to | Asked? |
| --- | --- | --- |
| The owner's own Pochi la Biashara | The business | No |
| A till or paybill the owner said is the business's | The business | No |
| Money sent to the owner's number | Personal | No, unless the owner asks for it (decision 15) |
| Anything else unclear: an unknown till, a bank, a reversal | As today: "Was this for the business?" | Yes |

Money out keeps today's rule: personal unless answered "Business cost". "Always do this" still learns per person, and any payment can still be switched (section 2).

**Where its balance lives.** That depends on what Safaricom reports, and changes nothing else:

- *The channel has a balance of its own* (a Pochi purse, a till's takings held in the till). It is an account of its own, in the business, checked against its own reported balances. Moving its money to the wallet is a transfer between spaces (section 6): the account goes down in the business and the owner's account up; in Personal, M-Pesa and Held for the stall go up. Calling it pay instead makes it Pay yourself.
- *The money lands in the wallet* and the message prints the M-Pesa balance. The payment goes to the business without a question, and Personal keeps only its balance record, with the postings of section 2.

**Pochi: a member of the family, or a channel.** 0026 lists the owner's Pochi as a member of the M-Pesa family, an asset read in the business (its owner decision 7). That is right if Pochi keeps its own balance, which is what Safaricom sells it for: the M-Pesa pack's own fixture carries Safaricom's line "Separate personal and business funds through Pochi la Biashara on *334#". No pack rule, fixture or document shows a Pochi receipt or a Pochi balance, so this is not confirmed. Proposed: Pochi is its own member, `familyOf` the wallet, living in the business. If the samples show money into Pochi printing only the M-Pesa balance, Pochi is a channel on the wallet and takes the second case. Paybill collections normally go to the organisation's own account, not the wallet; they reach the line as a payment from the paybill (the statement pack's `business_payment`), which is a transfer between spaces. A till is one or the other, by what its messages show.

**How the app will know the channel.** Today it cannot (Context). Rules come from the owner's own messages, rewritten as made-up fixtures, never guessed (0013, 0026 section 4). Samples needed:

1. Money received into the owner's Pochi, with whatever balance it prints.
2. Pochi to the M-Pesa wallet, and a Pochi balance enquiry.
3. A customer's payment into the owner's till, as the till's owner receives it, and any till balance.
4. Moving a till's takings to M-Pesa or a bank, or a till that pays straight into the wallet, if offered.
5. A paybill's collection notice to its owner, if there is one, and its payment to the wallet or a bank.
6. The statement rows for each, from the receiver's side.

Until a rule exists, such messages are asked about, as today.

**How the person names their till or paybill.** In Business or personal, under the line: "Your business's till or paybill", typed, or read from the M-PESA QR code the owner already adds for invoices. When a message names a till or paybill the app has not seen, it asks once: "Is till 123456 Vegetable stall's?". Pochi needs no number: there is one per line, offered on its first message (0026 decision 5). `RoutingRules` keeps the channels by line (0026 section 6 adds routing by line).

**The numbers.** One line with one reported balance; the wallet holds KES 37,819, all the owner's. The till pays into the wallet here, and so does Pochi in the first table. Fees left out.

| Step | The wallet says | Personal | Vegetable stall | Check |
| --- | --- | --- | --- | --- |
| 1. KES 650 sale into the stall's Pochi | 38,469 | M-Pesa 38,469. Held for the stall 650. Money in 0 | Owner's account 650. Sales 650 | 37,819 + 650 |
| 2. KES 1,200 into the stall's till | 39,669 | M-Pesa 39,669. Held for the stall 1,850. Money in 0 | Owner's account 1,850. Sales 1,850 | 37,819 + 1,850 |
| 3. The owner sends a friend KES 500 | 39,169 | M-Pesa 39,169. Held for the stall 1,850. Money out 500 | Unchanged. Profit 1,850 | 37,319 + 1,850 |

Personal's Home: On hand KES 39,169, "KES 1,850 of this is Vegetable stall's", safe to spend worked from KES 37,319; money in 0, money out 500. Personal's Activity shows one payment, the KES 500 to the friend. The stall's Today: two sales, KES 1,850, which are in its Activity and nowhere else; Money: "On your M-Pesa KES 1,850"; profit KES 1,850. Together: 39,169, what the wallet holds.

If Pochi keeps its own balance (proposed), the same three steps give: the wallet 38,519 (37,819 + 1,200 − 500) and Pochi 650. Personal: M-Pesa 38,519, Held for the stall 1,200, money out 500. The stall: Pochi 650, On your M-Pesa 1,200, Sales 1,850. Together 39,169 again, each reported balance checked in the space where its account lives. When the owner moves the 650 from Pochi to the wallet, the stall shows Pochi 0 and On your M-Pesa 1,850, and Personal M-Pesa 39,169 with 1,850 held for the stall. Nothing counts twice and nothing is lost.

### 5. What the person sees

**Personal**

- **Home.** On hand stays what the accounts hold, so it matches M-Pesa. Under it: "KES 650 of this is Vegetable stall's". Safe to spend, "left this month" and the forecast are worked from the rest. Money the stall owes the owner is shown, and is not added to what can be spent.
- **Activity, search and exports.** A payment switched to the stall is not in them. It is in the stall's only. Switched back, it returns here and leaves the stall's. A purchase paid with the stall's money shows here under its own category, with "Paid from Vegetable stall" (section 6).
- **Balances.** M-Pesa shows the wallet. The balance check adds up with no gap, because it counts the balance records; it shows them only as the total held for the stall, never as payments.

**The business**

- **Today and Insights.** The sale counts as a sale when the owner answers "Business sale", and it is in the stall's Activity, search and exports. Costs count as costs. Profit and the tax estimate follow.
- **Activity.** A payment from the stall's own account that the owner marked "This was for me" shows as Paid yourself, not as a cost (section 6).
- **Money.** A line beside Cash, M-Pesa and Bank: "On your M-Pesa KES 650" ("On your own accounts" when there is more than one). Below zero it reads "You have put in KES 350", and is not counted as money the business has.
- **Pay yourself.** "From" lists the business's own accounts and "Your M-Pesa (KES 650 is the stall's)". When real money moves, the sheet also asks where it lands.
- **Reports.** The balance summary shows "With the owner" or "Owed to the owner".

**Business or personal** reads: "M-Pesa: used for both. It stays in Personal; the stall keeps what is its own."

### 6. Money between the owner and the business

- **Pay yourself.** A business can always pay its owner. Pay yourself is already in the app, and it stays. It works on a shared line, where no money moves and the business's part of the line becomes the owner's (step 3), and between different accounts, such as the stall's KCB into the owner's M-Pesa (step 4). The agreed limits hold: on a shared line, no more than the business has on that line (decision 5); from an account of the business's own, what that account holds. It is never a cost and never counts in profit. In Personal it is "Salary from my business".
- **Put money in** is new, the reverse of Pay yourself. It is never income.
- **Real transfers between spaces** (a business bank to the owner's M-Pesa, or back) pass through the same pair of accounts, so each side can be recorded when its own message arrives. A message one bank sends about the other side of a transfer is kept with that bank's own payment. No account is made for an institution another space reads.

Pay yourself and Put money in are transfers between two spaces, like a transfer between two of your own accounts: each space shows its own side, money out of the business and pay into Personal. They are not one transaction shown twice.

**Spending the business's money directly.** The owner does not have to pay themselves first and then spend. They can pay for something personal straight from the business's account, till or Pochi (once their messages are read, section 4).

- *How it is marked.* On the business's own accounts a payment out is the business's, as today. The owner answers "This was for me": when the app asks "Was this for the business?", or on the payment later. "Always do this" learns it per person, as today.
- *In the business* it is "Paid yourself": money its owner took, the owner's drawing. The account goes down. It is never a cost, and never counts in profit or the tax estimate. Its row reads "Paid yourself, KES 800, from KCB". It does not say what was bought.
- *In Personal* it is the purchase, under its real category: "Food, KES 800, Mama Oliech's Kitchen, Paid from Vegetable stall". It is Personal's spending: it counts in money out, in the Plan's Food for the month, and in Insights. It adds nothing to Personal's money in: it is not Salary from my business or any other income. No Personal account goes down, so On hand does not change. Safe to spend counts the 800 as Food spending, like any other: what is left of the month's day-to-day budget is 800 less. Personal files the category the usual way, and the person can change it there.
- *Money in less money out.* For such a purchase it no longer equals what Personal's accounts did. Personal's Out goes up by 800, its In does not, and no Personal account moves, because the money came from the business. The difference is the business's money, and Personal shows it as "Paid from Vegetable stall" on each such row.
- *The books.* The business: KCB down, Paid yourself up. Personal: Food up, Paid by Vegetable stall up. Paid by Vegetable stall is the stall's money that paid for the owner's purchase; it takes the role that counts in neither money in nor money out (section 9), so it is never income. Each entry balances in its own journal. The two carry one link and are written as a pair (section 7).

**The mirror: a business cost paid from the personal line.** The owner pays the stall's supplier from M-Pesa, which lives in Personal, and answers "Business cost". That is a switch (section 2). The cost shows in the stall, "Paid from your M-Pesa", and counts in its costs and profit. Personal keeps only the balance record: M-Pesa down, Held for the stall down. It is no row and no spending in Personal.

**One rule for both.** A payment shows as what was bought in the space it was for. The space whose money paid keeps what its own books need, and shows only what is its own to show.

| Paid from | For | The space that paid | The space it was for |
| --- | --- | --- | --- |
| The stall's KCB, till or Pochi | The owner | The stall: a row, Paid yourself 800. Never a cost | Personal: a row, Food 800, "Paid from Vegetable stall". No account goes down |
| The owner's M-Pesa, which lives in Personal | The stall | Personal: no row. M-Pesa's balance, and "Vegetable stall owes you KES 400" (or, when the stall had money on the line, "KES 250 of this is Vegetable stall's", as in section 3) | The stall: a row, Stock and supplies 400, "Paid from your M-Pesa" |

The business gets a row and Personal does not because a business must show every shilling its owner takes, as Paid yourself, for its profit, its tax and its owner's pay. When the owner's line pays a business cost, Personal has nothing of its own to report: the money was the stall's, or the owner lent it to the stall, and the line under On hand already says which.

**Why this is not a transaction in both spaces.** Switching moves a transaction: it shows in one space, and the other keeps only a balance record. Beside Pay yourself and Put money in, a personal purchase with the business's money is the one case where each space has its own linked entry. The two entries are two different facts about one payment: the owner took KES 800 from the stall, and the owner bought lunch for KES 800. The stall shows the first and never Food. Personal shows the second and never a KCB payment or a balance going down. Neither is a copy of the other: change the category in Personal and the stall's entry stays as it is. Nothing counts twice: the 800 is not in the stall's costs or profit, and it is not taken off any Personal account. It counts once as spending, in Personal, and never as money in.

Switching it works as in section 2. "This was for the business" (the lunch was for the stall's helper) removes the Personal entry and turns Paid yourself into a cost. Switching a payment from the business's own account to Personal is the same as answering "This was for me".

**The numbers.** Made-up, starting again. The owner's M-Pesa lives in Personal and holds KES 37,819, all the owner's. The stall's KCB holds KES 10,000. It is the first of the month: Personal's money in and out are 0, and the stall has no sales or costs yet. Fees left out.

| Step | M-Pesa says | KCB says | Personal | Vegetable stall | Check |
| --- | --- | --- | --- | --- | --- |
| 1. Lunch at Mama Oliech's Kitchen, KES 800, paid from the stall's KCB. The owner answers "This was for me" | 37,819 | 9,200 | M-Pesa 37,819. Row: Food 800, "Paid from Vegetable stall". In 0, Out 800. On hand unchanged at 37,819. Food in the Plan 800 more, so the day-to-day budget left is 800 less | KCB 9,200. Row: Paid yourself 800. Costs 0. Profit 0 | 37,819 + 9,200 = 47,019, which is 47,819 less the lunch. The 800 counts once: as Food in Personal |
| 2. Stock from Example Suppliers, KES 400, paid from M-Pesa. The owner answers "Business cost" | 37,419 | 9,200 | M-Pesa 37,419. No row. In 0, Out 800. "Vegetable stall owes you KES 400". Safe to spend worked from 37,419 | Row: Stock and supplies 400, "Paid from your M-Pesa". "You have put in KES 400". KCB 9,200. Profit −400 | 37,419, all the owner's; the stall owes the owner 400. 37,419 + 9,200 = 46,619, which is 47,019 less the stock. The 400 counts once: as the stall's cost |
| 3. The lunch was for the stall's helper after all: "This was for the business" | 37,419 | 9,200 | The Food row goes. In 0, Out 0. Food in the Plan back to what it was. M-Pesa and On hand unchanged | Paid yourself 0. Row: Staff meals 800, a cost. Profit −1,200. KCB 9,200. "You have put in KES 400" | No balance changes. The 800 still counts once, now as the stall's cost |

After step 1, Personal's In less Out is −800 (0 less 800), but its M-Pesa did not move: it is 37,819 before and after. The 800 is the lunch the stall's money paid for: it is Personal's spending, and it came off the stall's KCB, not off any Personal account. Step 2 does not change that gap: the stock is no row in Personal, and M-Pesa's fall of 400 is matched by "Vegetable stall owes you KES 400". The stall's side adds up on its own: KCB 10,000 less the 800 it paid out is 9,200.

Had the owner first paid themselves KES 800 from KCB into M-Pesa and then paid for lunch from M-Pesa, the balances and the stall's figures would be the same: KCB 9,200, the stall's Paid yourself 800, M-Pesa 37,819 (37,819 + 800 − 800). Personal would differ: In 800 as Salary from my business, Out 800. Paid directly, Personal's In is 0 and its Out is 800.

### 7. Keeping the two sides in step

Two files cannot change in one step. Each linked pair carries the same link id on both sides, and is written so that doing it twice changes nothing. When a space opens, any pair with one side missing is finished or undone.

- **Restore.** Restoring one space from an older backup can leave the two sides apart. The check shows which entries are on one side only and offers to make the other side again.
- **Clear data and Delete space.** The owner's account is settled first, so Personal does not go on holding money for a space that is gone (decision 9 below).
- **Switching back.** "It was mine after all", or "This was for the business" on a personal purchase, reverses both sides of the pair in one repeatable write, so the transaction is never in both spaces or in neither.

### 8. Records people already have

People who answered "Business sale", or moved payments by hand, have copies in the business with the wallet's balance on them, and markers in Personal (`moved_transactions`, `set_aside_messages`). A repair puts each right:

1. For each marker, find the copy in the other space.
2. Put a balance record back on the account it came from (section 2): the amount, time, fee, M-Pesa code, the balance its message printed, the link and the space it went to. The copy holds everything needed; the message itself is not. Nothing is added to that space's lists.
3. Turn the copy into the linked entry: the same amount, time, name, category, and its links to sales, customers and suppliers. Its balance is dropped and it moves to the owner's account.
4. Remove the marker, and rebuild both journals (0018's rebuild).

Accounts in the business that only moves made, and that hold nothing of their own, are removed. Where the owner had typed an opening balance or a Set balance on one, the app asks once: "How much of your M-Pesa is Vegetable stall's today?", and offers what the linked entries add up to.

What the repair cannot settle is listed for the person, as 0018's migration does: a marker whose space was deleted or cleared (the payment goes back to Personal as the owner's own when its message is still on the phone), an account deleted since, and payments a hand-over copied into two spaces (those dated before the second space was given the line are offered for removal there). A locked copy of every space is kept for 30 days before the repair (0016).

### 9. Migration

- Each account gains balance records: what a switched payment leaves behind where its account lives (section 2). They are not `Transactions` rows, so no list, total, search or export can pick one up by mistake. `WalletWalk` walks an account's transactions and its balance records together.
- `Transactions` gains the link id and the space on the other side, for a switched payment and for each side of Pay yourself, Put money in and a personal purchase with the business's money. One account type is added for the owner's account.
- The journal gains three account kinds: held for a space (owed), the owner's account (owned), and paid by a space (Personal's other side of a purchase with the business's money). `roleOf` gains one role that counts in neither money in nor money out.
- `RoutingRules` records which space each account lives in, and the business's channels on a line (its Pochi, tills and paybills).
- The owner's Pochi becomes an `Accounts` row in the business with `familyOf` the wallet (0026), if it keeps its own balance.
- `moved_transactions` is emptied by the repair and no longer written.
- The journal is rebuilt in every space, and anything that does not reconcile is reported.

### 10. Stages

| Stage | Work | Tested by |
| --- | --- | --- |
| Before | Fixes that do not wait for this: balances read from the right space after a switch, one reader for each account, a backup restored only into its own space | The audit's tests for those |
| 1 | This decision and the owner's answers (done); mockups | |
| 2 | The books: the two accounts, the new role, the posting rules, balance records | The numbers in sections 3 and 6; random histories in which the two sides always agree |
| 3 | Switch: the transaction moves, its balance record stays; switching back; "Was this for the business?" uses it and makes the sale; a line that lives in Personal is read whichever space is open | The audit's failing tests for a shared line, which must pass; a switched payment found in exactly one space by every list, search and export, before and after switching back |
| Channels | Rules for money received through the owner's Pochi, till and paybill, from the owner's samples; Pochi as a member (0026); routing by channel; naming the till or paybill | Section 4's numbers; fixtures from the samples; a line with both kinds of money replayed for a month |
| 4 | Screens: Personal's Home, Activity and Balances; the business's Money, Today, reports, tax and exports; Ask | One test per figure that the new role is left out of money in and out; one test per list, search and export that no transaction shows in two spaces |
| 5 | Pay yourself and Put money in on the owner's account; "from" and "into" asked apart; "This was for me" on the business's own accounts, with its linked purchase in Personal | Steps 3 and 4 of section 3; section 6's numbers, including Personal's In staying 0 for a purchase with the business's money |
| 6 | The repair of records already there, with its report | A replay of moved payments, opening balances and corrections from the audit |
| 7 | Giving a line to a space as a dated event; transfers between spaces by bank and M-Pesa | Step 7; the bank and M-Pesa cases from the audit |

Stages 2 to 6 are built and tested in this order and reach people in one release. Channels can be built once stage 3 is in and the samples are in hand; until then those payments are asked about. Switching (stage 3) and the repair (stage 6) must arrive together, so that nobody has records of both kinds.

## Alternatives

- **B. Shares of the line.** Each space keeps its own share of the wallet as an M-Pesa account, and the shares add up to the reported balance. The example gives the same amounts as A: after step 1 the shares are 37,819 and 650; after step 3 they are 38,019 and 50. The differences are in what it costs to keep them true. A reported balance can only be checked by adding two spaces' records, for every message, which 0016 and 0018 rule out. A gap belongs to neither share until someone says whose it is, so it needs a third place. In the KES 1,000 case the business's share is an M-Pesa balance 350 below zero with no Fuliza. Fuliza owed on the line would have to be shared out. The owner must type an opening share. At a hand-over the shares merge and the owner's part still needs an owner's account. On screen, each space shows an M-Pesa figure that no message ever printed, so Personal's M-Pesa no longer matches the phone. Cost: all of A's work and more. Rejected: harder to build and harder to trust.

  | Step (section 3) | Personal's share | The stall's share | What goes wrong |
  | --- | --- | --- | --- |
  | 1. Sale 650 | 37,819 | 650 | Personal's M-Pesa no longer shows what the phone says |
  | 2. Cost 400 | 37,819 | 250 | |
  | 3. Pay yourself 200 | 38,019 | 50 | |
  | 4. KCB to M-Pesa 2,500 | 40,519 | 50 | |
  | 5. The wallet is 200 short | 37,819 or 37,869? | 50 or −150? | The gap is nobody's until someone says |
  | 6. Old cost 4,000 moved | 42,019 | −3,950 | An M-Pesa share below zero with no Fuliza, and past shares rewritten with no reported balance to check them |
  | 7. Line handed over | merged into the stall's 38,069 | | The owner's 38,019 still needs an owner's account |
- **C. Paid to the owner at once.** Every business payment on the owner's line is settled the moment it is answered: the sale is paid straight to the owner, and a cost is money the owner put in. After steps 1 and 2 the stall shows Sales 650, costs 400, Paid yourself 650 and "you have put in" 400; Personal shows income 650 and 400 put into the stall. Nothing is held for anyone and no shilling counts twice. Cost: small, with no running account and no new line on any screen. But Personal's income becomes the stall's takings, safe to spend treats the stall's float as the owner's own, and no screen can say how much of the wallet is the stall's. Rejected as the rule. It is what A gives when the owner pays themselves everything the stall holds.
- **D. Keep moving, without the balance.** Drop the reported balance when a payment is moved. The business's M-Pesa then adds up to what it was given (650), and its opening balance and corrections hold. Cost: the smallest, four fields left behind on a move, and no screen changes. But Personal's balance still rests on records it no longer has, the two spaces together count the 650 twice once Personal's next message comes, and Pay yourself, costs paid from the line and a hand-over stay as they are. Rejected as the answer; usable as a stopgap before stage 3 (decision 13).
- **Read the other space's records instead of linking.** Keep the payment only in Personal and let the business look it up there. One copy, nothing to keep in step. But a business would no longer be whole on its own: its backup, restore and kept copy would miss its sales. Rejected (0016).

## Consequences

- One sale no longer breaks any balance. Personal's reported balances add up from its own records, and the business's figure cannot be overwritten by a message.
- Money in and out, profit and the tax estimate count each payment once, in the space it belongs to. Money the owner puts in or takes out is never income or a cost.
- Pay yourself on a shared line is recorded at once on both sides, as the spec promised. A business can always pay its owner, within the agreed limits.
- The owner can spend the business's money on themselves directly. It is Paid yourself in the business and spending only in Personal: it counts in Personal's money out and its Plan, never in its money in. So Personal's money in less money out no longer always equals what its accounts did.
- Each transaction shows in one space only, and can be switched back and forth. Where a switched payment's account lives, a balance record stays: Personal's file still holds the amount, time, fee, M-Pesa code and printed balance of a business sale on the owner's line, but not the other person's name, which is only in the business's file and backup.
- 0016 changes: a payment read from a message or statement leaves a balance record behind when it is switched to another space. 0018 gains two accounts, one role and balance records. The spec's "a transaction lives in one space only" holds on screen, and gains "an account lives in one space only".
- Every screen that adds up money must leave the new role out. A missed one would count the business's money as the owner's again, so each gets a test.
- Two files change for one answer. The link id, the repeatable write and the check carry that weight.
- The repair rewrites records in two spaces. It is the riskiest stage, which is why it keeps a copy and reports what it could not settle.
- On one line with the business's own channels, most business money is sorted without a question once the rules exist. Those rules wait on the owner's samples.
- Not settled here: one line shared by two businesses, two M-Pesa lines read as one (0026), and a business with two owners.

## Owner's decisions

The owner approved the recommendations and the proposed answers, and added three rules (7, 19 and 20):

1. **The model.** Option A: an account lives in one space. A payment that belongs to another space is switched there, and the account's own space keeps a balance record against Held for that space, linked to the entry on an owner's account.
2. **The words.** In Personal: "Held for Vegetable stall", and "Vegetable stall owes you" below zero. In the business: "On your M-Pesa", and "You have put in" below zero. In reports: "With the owner" and "Owed to the owner".
3. **Personal's On hand.** It stays what the accounts hold, with "KES 650 of this is Vegetable stall's" under it. Safe to spend leaves that part out.
4. **The business's money.** What is on the owner's accounts counts as money the business has, on its own line. Money the owner has put in is shown and not counted as money.
5. **Pay yourself on a shared line.** No more than the business has on that line. To take more, pay from another business account.
6. **Fuliza.** When a business cost on the owner's line is covered by Fuliza, the debt and its charges stay the owner's unless a charge is marked for the business.
7. **Switching a transaction to another space (the owner's addition).** Any transaction can be switched to another space, and back. From then on it shows only in that space: its Activity, totals, reports, search and exports. No transaction is a row in two spaces. One read from a message or statement leaves only a balance record where its account lives (section 2). One typed by hand moves outright.
8. **The repair.** It runs once on upgrade, keeps a locked copy of each space for 30 days, shows what it changed, and asks one question for each business with a shared line.
9. **Deleting or clearing a business that still has money on the owner's accounts.** The app asks. "It is mine now" files it as pay from the business. "Keep it apart" leaves it held for the business until the owner settles it.
10. **Giving a line away.** The whole wallet goes with the line, and the owner's own part shows as money put in, with a prompt to move it out first.
11. **Older payments on a shared line.** A "Sort older payments" list with no time limit.
12. **Other kinds of space.** The same pair of accounts works between any two spaces. Personal and a business are built first, then Household and the rest with their own words.
13. **A stopgap.** Until stage 3 reaches people, a moved payment leaves its balance behind (alternative D). It stops a business showing the owner's whole wallet. Personal's side stays as it is today.
14. **Channels decide first.** On a line used for both, money into the owner's Pochi and into a till or paybill the owner named goes to the business with no question; money sent to the owner's number is personal.
15. **Customers who send money to the owner's number.** Personal by default, with "Customers also send money to my number" in Business or personal, which brings back "Was this for the business?" for those payments.
16. **Pochi.** Its own member of the M-Pesa family, with its own balance, living in the business, as 0026 says, if the samples show a Pochi balance; otherwise a channel on the wallet.
17. **Naming a till or paybill.** Typed, or read from the M-PESA QR code already added for invoices, and asked once when a message shows a number the app has not seen.
18. **Samples.** The owner collects the six kinds of message in section 4 from their own phone. They are rewritten as made-up fixtures before any rule is written.
19. **Pay yourself (the owner's addition).** A business can always pay its owner with Pay yourself, on a shared line or between different accounts, within the limits in 5 (section 6).
20. **Spending the business's money directly (the owner's addition).** The owner can pay for something personal straight from the business's account, till or Pochi, by answering "This was for me". It is Paid yourself in the business, never a cost, and the purchase under its own category in Personal, "Paid from Vegetable stall", with no Personal account going down (section 6).
    *The owner's answer, 21 November 2026:* in Personal it is spending only. It counts in Personal's money out, its Plan category and Insights, and adds nothing to Personal's money in: it is not Salary from my business or any other income. No Personal account goes down, so On hand does not change, and Personal's money in less money out no longer equals what its accounts did for such a purchase. In the business it stays Paid yourself.

## Screens (approved 21 November 2026)

The owner approved the mockups for this decision, with these choices:

1. The action is named "Switch to another space" (today's "Move to another space" is renamed).
2. The tag on the business's rows for money on the owner's line is "Your M-Pesa" (or the account's own name).
3. On the business's own accounts, "Was this for the business?" is asked only when it is unclear.
4. "No" on "Is till 123456 Vegetable stall's?" falls back to today's business-or-personal question.
5. Money in switched to a business is filed as Sales by default, and can be changed.
6. Everything else as drawn: the repair sheet's wording, the Kind menu (Till or Paybill) when naming a number, and the tiles' short lines.
