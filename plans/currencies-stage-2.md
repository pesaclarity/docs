# Plan: own transfers across currencies (currencies stage 2)

When a person moves money between two of their own accounts in different currencies, such as KES 13,050 out of a shilling account and USD 100.00 into a dollar account at the same bank, or onto a card wallet, the app must see one transfer, not spending on one side and income on the other. It builds on [0009](../decisions/0009-base-currency.md), [0014](../decisions/0014-rates-of-the-day.md) and [0018](../decisions/0018-double-entry-journal.md), and on the [journal plan](journal.md), which leaves multi-currency own transfers to this stage.

All names, numbers and rates here are made up.

## Where things stand

- **Stage 1 shipped.** Each amount keeps the currency its message prints, rates of the day are kept (`rate_snapshots`), a transaction can carry the rate the person got, accounts at one bank are told apart by masked number, and a card payment in another currency is merged with the account's own debit for it when the two are within 15 minutes and within 10% of the day's rate (`findCardTwin`, `cardAmountsAgree`).
- **Own transfers pair in one currency only.** `findOwnTransfers` (`lib/src/domain/ledger/own_transfers.dart`) groups entries by currency, so KES out and USD in never pair. Pairing runs on amounts as stored (`pairOwnTransfers` reads entries with no converter).
- **But matching by hand already crosses currencies, by accident.** "Was this your own money?" (`transferSuggestionsProvider`) reads entries converted into the base, so USD 100.00 in reads as KES 12,920 and `suggestTransferPartners` can offer it for KES 13,050 out as "plus a 130 fee". Matching it saves the pair, and the journal then leaves KES 13,050 and USD 100.00 in Money in transit that never clear each other. Nothing else is wrong with that pair: Activity shows one line and totals leave it out.
- **The journal stores each entry in one currency.** `postJournal` posts every transaction in its own currency, and the stored postings of an entry share one currency; figures convert postings into the base when they are read, each at the rate of its own day (`JournalRepository.entries`). So an account in USD is worth, in the base, the sum of its amounts at the rates of their days: it does not move when today's rate moves.
- **Journal stage 3 is being built** (fixes as entries, Money in transit clearing with fees). It changes `own_transfers.dart` and the journal's posting code, so this stage does not touch them until it lands.

## 1. The matching rule

The rule is `findCrossCurrencyTransfers` in `lib/src/domain/ledger/cross_currency_transfers.dart`, with `suggestCrossCurrencyPartners` for matching by hand. It runs after `findOwnTransfers`, on what that leaves unpaired. It takes its sides from the same-currency rule and its amount check from the card-payment rule, so the app has one way of thinking: a time window, a tie between the two accounts, and an amount check, with the person's word winning.

### Which entries can be a side

As for one currency: unpaired, not a correction, a `sent`, `paybill` or `bank` movement out or a `received` or `bank` movement in, not in a non-transfer category, and not typed in by hand unless the person said where it went. Two more conditions:

- Each side is in the currency of its own account. A card payment abroad on a shilling account is not: it belongs to the card-payment rule.
- The two currencies differ, and the accounts differ.

### The worked example

Saturday 21 November 2026. The day's rate is 1 USD = 129.20 KES.

| | Account | Time | Amount |
| --- | --- | --- | --- |
| Money out | KCB ···4321 (KES) | 10:00 | KES 13,050.00 |
| Money in | KCB ···8765 (USD) | 10:02 | USD 100.00 |

- At the day's rate, KES 13,050 buys USD 101.01. USD 100.00 came in: 1.0% less.
- The rate the person got is the ratio of the two amounts: 13,050 / 100 = 1 USD = 130.50 KES.
- The cost of the exchange is the difference from the day's rate: KES 13,050.00 less USD 100.00 at 129.20 (KES 12,920.00) = **KES 130.00**.
- Both accounts are at KCB, two minutes apart, and the rate is within 5% of the day's: they pair by themselves.

### When a pair is made by itself

| What ties the two | Time apart | Rate against the day's |
| --- | --- | --- |
| Both accounts at the same institution (two KCB accounts, two wallets of one card provider) | 30 minutes, as for one currency | within 5% either way |
| One message prints the other account's number (a PesaLink to the person's own dollar account, an M-Pesa paybill payment for it) | 3 days, as a numbered transfer waits today | within 10% either way |
| The person named the other account ("To another of my accounts") | 3 days, as a named transfer waits today | within 10% either way |
| Nothing but time and rate (two institutions, nothing named) | never by itself: only offered by hand | |

- **The rate is checked in the money in's currency**, against what the money out bought at the rate of the day the money left. In the example, USD 95.96 to USD 106.06 pairs by itself at one bank; USD 90.91 to USD 111.11 pairs when named or numbered.
- **Why 5% and 10%.** Banks sell dollars to their own customers at 1 to 3% from the mid rate, and the free rates service gives the mid rate. A card payment is allowed 10% because both of its messages are on one account and the 15-minute window is tight. A transfer has two accounts, and a wrong pair hides real income and real spending at once, so without a stronger tie it gets half of that. A number or the person's word is as strong a tie as one account, so it gets the card's 10%.
- **Either way.** More can come in than the day's rate gives (the day's rate was fetched early, or the bank ran a promotion), so the check is both ways, as the card rule is.
- **Different from one currency.** Same-currency pairs need no tie within 30 minutes, because equal amounts (or the outflow larger by a fee) rarely happen by chance. Across currencies any two amounts give some rate, and 5% is wide enough for chance pairs across banks, so a tie is needed. Same-currency pairs never allow more in than out; across currencies, more in is allowed.

### Fees and charges

- **A fee the money out's message prints** is on top of its amount, as today (`feeMinor`), and the rate is worked out without it. KES 13,050 and a KES 50 fee for USD 100.00 is still 130.50, at a cost of KES 130 plus the KES 50 fee.
- **A fee folded into either amount** cannot be told from the rate: it shows as a worse rate. KES 13,350 for USD 100.00 (KES 13,050 plus KES 300 taken in the same debit) is 133.50, 3.3% worse than the day's: it still pairs at one bank, at a cost of KES 430.
- **A charge in its own message** ("KES 50 charged for a transfer") is a `charge`, never a side, and counts as a fee as today.
- **A small move with a flat fee** can fall outside 5%: USD 10.00 for KES 1,400 (KES 1,300 plus a KES 100 charge) is 8% worse and is only offered by hand, unless the person named the account.
- **A fee on the money in** (a receiving charge on the dollar account) comes off what arrived and counts as a fee; the rate uses the amount the message says was credited.

### Two candidates

- **A number first, then the person's word, then the same institution.** A paybill payment that names the person's Equity dollar account pairs with that account even when a KCB dollar credit arrives closer in time.
- **Then the closer rate fit** (within 5% before within 10%), **then the closest in time, then the rate nearest the day's**, then by id, so the result never depends on the order entries are read in.
- **Each entry is used once.** Two moves of KES 13,050 and KES 26,100 with USD 100.00 and USD 200.00 arriving each find their own side: the amounts fit different rates.

### A partial match

- **Money in for part of the money out** (KES 26,100 out, USD 100.00 in: 1 USD = 261 KES) does not fit the rate and is never a pair.
- **Money that arrives in two parts** (USD 60.00 and USD 40.00 for KES 13,050) is not paired and not offered: one out pairs with one in, as for one currency (decision 5).
- **The rate fits but the time does not** (the dollars arrive the next morning, no tie but the bank): offered by hand, never paired by itself.

### A reversal

- **A reversal message is never a side.**
- **A side the bank undid is not a side.** KES 13,050 out and then KES 13,050 back as a reversal on the same account means nothing left to arrive in dollars, so neither is offered (`reversedEntries`: each reversal takes the latest entry before it on that account, the other way, with the same amount and currency, up to 3 days earlier).
- **A pair whose side is reversed later** is unmatched when the reversal arrives, so both count as they were and the reversal cancels its side (wiring, after stage 3).
- **A move the person undoes themselves** (USD 100.00 sold back for KES 12,750) is a second transfer the other way; its cost is counted again. Two moves cost twice.

### A missing rate for that day

- **No rate for the day** (rates never loaded, or a currency the service does not list): nothing pairs by itself, not even with a number or the person's word, because nothing checks that the amounts are one move. The pair can still be matched by hand: it is offered as "no rate for that day", and the cost is "not known".
- **Only an approximate rate** (a day before any rate was kept, so the oldest kept rate stands in): amounts that fit it pair as usual, but amounts that do not fit it are still offered by hand, because the real rate of that day may have been different. The cost is "not known".
- **The person's own rate** for the currency, when set, wins over the kept rates, as everywhere else.

### Never an own transfer across currencies

- Two sides on one account (a card payment and its debit: the card rule's).
- A side that is not in its account's currency.
- Two amounts in the same currency (the same-currency rule's), or two the same way.
- Till payments, withdrawals, airtime, Fuliza, loan repayments, corrections.
- A side in a non-transfer category, or one already paired.
- A pair the person unmatched (`declined_transfers`).
- Money in that names another bank account as where it came from, money out that names another account as where it went, or a side the person named for another account.

## 2. What is stored

- **Both transactions stay as they are**, each with its own amount and currency. Pairing sets `transfer_pair_id` on both, as for one currency. No new table and no schema change.
- **The rate the person got is the ratio** of the two amounts (`CrossCurrencyTransfer.rateGot`) and is worked out when shown; nothing writes `fx_rate_currency` or `fx_rate_micros` on either side. Writing the rate got onto the dollar side would make it convert at 130.50, hide the cost inside the dollar balance, and make the dollar account's later payments count at a rate no message gave.
- **The difference from the day's rate is the cost of the exchange.** It is not stored: the journal posts it (section 3), worked out again whenever the journal is reposted, so a rate the person sets later fixes it everywhere.
- **A rate the person set on one side** (Use the rate I got, from stage 1) still wins for that side, and the cost is worked out with it.

## 3. The journal

Each stored entry keeps one currency. A cross-currency transfer has two currencies, so it is three journal entries: each side as today, and one exchange entry per currency when the two are matched. Money in transit is kept per currency, and a **Currency exchange** account (a trading account, as GnuCash and other multi-currency ledgers keep) holds what was sold in one currency and bought in the other, so every entry sums to zero in its currency. The cost goes to a new **Exchange cost** account, an expense.

For the example:

| Entry | Currency | Dated | Debit | Credit |
| --- | --- | --- | --- | --- |
| 1. Money out (as today) | KES | 10:00 | Money in transit 13,050.00 | KCB ···4321 13,050.00 |
| 2. Money in (as today) | USD | 10:02 | KCB ···8765 100.00 | Money in transit 100.00 |
| 3a. The exchange | KES | 10:02 | Currency exchange 12,920.00, Exchange cost 130.00 | Money in transit 13,050.00 |
| 3b. The exchange | USD | 10:02 | Money in transit 100.00 | Currency exchange 100.00 |

- Entry 1 sums to zero in KES, 2 in USD, 3a in KES (12,920 + 130 − 13,050) and 3b in USD.
- Money in transit is back to zero in both currencies.
- Currency exchange holds KES 12,920.00 and USD −100.00, which is zero in the base at 129.20.
- **The exchange entries are dated when the money arrived**, and the cost uses that day's rate, so when the two sides are days apart every account still clears exactly in the base (the dollar side and 3b convert at the same day's rate). When the base is a third currency (a person who counts in EUR moves KES into USD over two days), the shilling side can be worth a little more or less in the base on the second day; that move in value stays in Money in transit's base reading and shows in reports as "Moves in value at the day's rates". Per currency, transit is zero.
- **A fee the message printed** posts as today: entry 1 becomes KCB ···4321 credit 13,100.00, Money in transit 13,050.00 and Fees 50.00.
- **The cost is in the money out's currency.** Money out in USD and in in KES (USD 100.00 sold for KES 12,750) posts 3a in USD: Currency exchange 98.68 and Exchange cost 1.32 against transit 100.00 (KES 12,750 at 129.20 is USD 98.68); in the base, Exchange cost is about KES 170 (USD 1.32 at 129.20 is KES 170.54).
- **More in than the day's rate gives** makes Exchange cost negative for that entry: a gain at the day's rate, never income.
- **No rate for the day, or only an approximate one**: 3a posts the whole money out to Currency exchange and nothing to Exchange cost, since the cost is not known. The dollar side converts into the base however it does today (left out of totals with no rate, "Not in totals yet").

**An unmatched half while it waits.** A side the person named (or one whose message numbers the other account) posts entry 1 alone: Money in transit holds KES 13,050.00 and shows as money in transit, not spending, as a same-currency named transfer does after stage 3. A side nobody named posts as it reads today (spending, or income, or Not yet sorted for money in from no one named). When the other side arrives and matches, entries 2, 3a and 3b post and the journal reposts the first side as a transfer. Unmatching reverses 3a and 3b and reposts both sides by their own meaning (append-only, 0018).

## 4. What every figure shows

Before and after the example, for someone who counts in KES with KCB ···4321 at KES 50,000.00, M-Pesa at KES 8,000.00, an empty dollar account and no rainy day fund:

| Figure | Before | After | Why |
| --- | --- | --- | --- |
| KCB ···4321 | KES 50,000.00 | KES 36,950.00 | |
| KCB ···8765 | USD 0.00 | USD 100.00, shown ≈ KES 12,920 | at the rate of its day |
| On hand | KES 58,000 | KES 57,870 | down by the exchange cost only |
| Safe to spend today (a weekend day) | KES 810 | KES 810 | the cost is not day-to-day spending; it moves only when On hand is what limits it |
| Left this month | KES 11,240 | KES 11,240 | the same |
| End of month at this pace | KES 113,670 | KES 113,540 | follows On hand |
| Activity, Out this week | KES 7,820 | KES 7,950 | the cost counts with fees, as transfer fees do |
| Activity, In this week | KES 2,500 | KES 2,500 | a transfer is never income |

Without this stage (today), the same two messages would add KES 13,050 to Out and KES 12,920 to In, with On hand the same KES 57,870.

- **On hand and safe to spend.** A transfer moves money between accounts, so only the cost changes On hand: KES 13,050 out, USD 100.00 in at 129.20 (KES 12,920), KES 130 less. A fee the message printed comes off as well. If the dollar account is a savings account (left out of On hand), On hand drops by the whole KES 13,050, as moving shillings into savings does today. Later moves of the rate do not change the dollar account's worth: each amount stays at the rate of its day (0014).
- **Activity "Out" and "In".** The transfer is in neither. The cost (and any fee) is in Out with fees.
- **Activity's line.** One line for the pair, as for one currency (see the mockups): the money out's amount with the money in's under it, each in its own currency. The transfer's sheet adds the rate the person got, the day's rate and the cost.
- **Plan.** Categories do not move: the transfer has no category and the cost is a fee.
- **Insights.** Not spending or income. The cost shows with fees ("Exchange cost") in What changed, Leakage (fees) and the Health Index's fee indicator, as bank charges do.
- **Business figures.** In a business space the cost is a cost in profit and loss, beside bank charges; Currency exchange is a clearing account that is zero at the day's rates and shows on the balance sheet only when a move in value is left (third-currency base). Books, Money and the business report show the pair once, as for one currency.
- **Reports and export.** The PDF report counts the cost with fees. Exports keep both rows with their own amounts and currencies, as today, and mark both as an own transfer.
- **Money in transit.** Shown per currency in its own currency; a waiting dollar side shows "USD 100.00 on its way" rather than a base figure that could move.

## 5. Matching by hand and undoing a match

Everything the app has for one currency, with the rule's suggestions added:

- **"Was this your own money?"** on a transaction's sheet lists same-currency partners (`suggestTransferPartners`) and cross-currency ones (`suggestCrossCurrencyPartners`): unpaired, the other way, another currency, up to 3 days apart, within 10% of the day's rate, best fit first, then closest in time, three at most. Each shows the partner in its own currency and the rate the two give: "KCB ···8765, USD 100.00 in · Today 10:02 · 1 USD = 130.50 KES". With no rate for that day every candidate in the window is offered, marked "No rate for that day". **Match** saves the pair as today (`match`, which also clears an earlier decline).
- **The same-currency suggestions stop crossing currencies.** `transferSuggestionsProvider` reads converted entries; it must compare entries in their own currencies (`originalCurrency`) so a dollar side is never offered as a shilling one with a "fee". This is a small change in `own_transfers.dart` and the provider, after stage 3.
- **"To another of my accounts"** works across currencies: choosing the dollar account makes the shilling side wait for USD (the named tie: 3 days, within 10%).
- **Unmatch** on the transfer's sheet works as today: the pair goes into `declined_transfers` and is never paired again by the rule, both sides count as they were, and the journal reverses the exchange entries.

## 6. What it needs from the message readers

The `ujumbe` packs (`ujumbe-parsers`) already read foreign-currency account messages: KCB's account alerts and A/C debited or credited messages print the currency with the amount and the masked own account; I&M's account alert reads EUR and USD; Equity reads money into a USD account; Standard Chartered reads a USD deposit. Fixtures exist for each (`kcb` "USD account debited", `im_bank` "EUR account credited", `equity` "receive into a USD account", `standard_chartered` "USD deposit"). Loop, Co-op, Airtel Money and M-Pesa read KES only.

What no pack reads yet:

- A message that names **both currencies or the rate** of a move between the person's own accounts ("KES 13,050.00 debited for USD 100.00 at 130.50"). With one, the app could pair by the message alone, with no rate check.
- The **other account's number** in a move between two accounts at one bank (it would give the number tie).
- **Equity's money out of a USD account**, and **Loop or any card wallet in another currency** (topping up a dollar wallet from shillings).

Sample messages needed (with every name, number and reference replaced by the person sending them): both messages of a KES to USD move between two accounts at KCB, Equity, I&M, Co-op and Standard Chartered; both messages of a USD to KES move back; a top-up of a dollar card wallet (Loop or another) from shillings; an M-Pesa paybill payment into a bank's dollar account and the bank's credit message for it; a reversal of a failed currency move; and any charge message sent with these moves.

## 7. Build order

Each step ships on its own with `tool/check` passing.

**Now, before journal stage 3** (touches nothing stage 3 owns):

1. **The rule, pure and not wired in.** *Done on `feat/currency-transfers`.* `findCrossCurrencyTransfers`, `suggestCrossCurrencyPartners`, `reversedEntries`, `dayRatesOf` (the app's rates as the rule needs them) and `CrossCurrencyTransfer` (rate got, cost against the day's rate, cost in any currency), with tests for every case in section 1.
2. **Sample messages and pack rules** in `ujumbe-parsers` (section 6), with fixtures. Independent of the app.
3. **The mockups** approved by the owner (section 4's figures, the Activity line and sheet, the waiting half, matching by hand).

**After journal stage 3 lands** (it changes `own_transfers.dart` and the posting code):

4. **Pairing runs the rule.** `pairOwnTransfers` runs `findCrossCurrencyTransfers` after `findOwnTransfers`, with each account's currency, number and institution and the rates of each day; `pairHistoryOnceProvider` runs once more over the whole history (version 2).
5. **The journal.** Currency exchange and Exchange cost book accounts; the exchange entries when a pair is in two currencies; Money in transit read per currency; reversing them on unmatch. Invariant test: every entry sums to zero per currency, and Currency exchange is zero in the base at the day's rates for a pair on one day.
6. **Matching by hand.** Cross-currency suggestions on the transaction sheet; same-currency suggestions stop crossing currencies.
7. **Figures.** Exchange cost counted with fees in `LedgerTotals`, Insights, the Health Index, business reports and the PDF report.
8. **Screens and copy** as the approved mockups.
9. **The replay test** gains a month with a KES to USD move at one bank, a named move to another bank two days later, one with no rate for the day, and a reversed one; every account must match its reported balances and On hand must be the sum by hand.
10. **Docs**: README (Currencies, Balances), the product spec's currency paragraph, and this plan marked as shipped.

## Decisions for the owner

1. **How close to the day's rate.** Pair by itself within 5% at one institution and 30 minutes apart, and within 10% with a number or the person's word and 3 days apart; never by itself with no tie. Or another pair of limits.
2. **What the cost counts as.** The difference from the day's rate counts with fees as "Exchange cost" (in Out, Insights and Leakage), recommended; or it only shows on the transfer and in On hand, counted nowhere else.
3. **A gain.** When more came in than the day's rate gives, the cost is negative and lowers fees, never income (recommended); or it counts as zero.
4. **No rate for the day.** Never pair by itself, even when named or numbered (recommended, the rule as built); or pair a named or numbered side with no rate check.
5. **Money that arrives in two parts.** Not paired and not offered for now (recommended); or allow matching one out to several ins by hand later.
6. **The Activity line and sheet.** The money out's amount with the money in's under it on the line; the rate got, the day's rate and the cost on the transfer's sheet, as in the mockups.
7. **When the exchange is dated.** When the money arrived, with the cost at that day's rate (recommended, so every account clears in the base); or when it left.
8. **Today's accidental cross-currency matches.** Stop the same-currency suggestions from offering other currencies in step 6 (recommended), or as a small fix before stage 3 lands, in agreement with the stage 3 worker since it touches `own_transfers.dart`.
