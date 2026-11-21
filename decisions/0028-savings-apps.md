# 0028. Savings apps such as Chumz

Date: 2026-11-21 · Status: Accepted · Builds on [0018](0018-double-entry-journal.md), [0026](0026-mpesa-family-of-accounts.md) and [0016](0016-spaces-kept-apart.md)

Accepted by the owner on 21 November 2026, with the mockup ("go with chumz") and the four additions below. Built in `pesaclarity-app` on `feat/chumz-savings` (domain, data and rules) and `feat/chumz-screens` (screens), not yet merged.

## Context

Chumz is a goal-saving app by Moneto Ventures; the money is held by Nabo Capital. People pay in through the M-Pesa paybill that M-Pesa prints as `NABO CAPITAL LTD`, with one account code per goal. Before this decision:

- **Payments into Chumz were spending.** A paybill payment is an outflow with a payee; nothing knew Nabo Capital as savings, so every pay-in lowered safe to spend and counted in the month's spending.
- **Money back was income.** A withdrawal arrives as money received from `NABO CAPITAL LTD` and read as money in, or as "Gift, loan or repayment?".
- **No balance.** Chumz sends no SMS: not for interest, not for its balance. The app had nowhere to keep what the person owns there.
- **Goals were typed.** A Pesa Clarity goal's saved amount was what the person typed, even when the money sat in a Chumz goal.
- 0026 makes M-Pesa products members of the M-Pesa family. Chumz is not one: Safaricom does not hold its money, and its only link to M-Pesa is the paybill.

## Decision

A **savings app** the app knows is kept as an account of the person's own, fed from M-Pesa. Chumz is the first; another app, fund or SACCO paid through a paybill is one more entry in a registry (`savings_product.dart`).

1. **An ordinary savings account.** Chumz is an `Accounts` row of a savings type (money market), marked with its product. It is left out of On hand and safe to spend and counts in net worth, as savings accounts already do.
2. **Every payment keeps its M-Pesa row and gains a mirror on Chumz**, paired with it as an own transfer: never spending, counted once, the M-Pesa fee still a fee, Money in transit at nothing at every moment (0018). Money back is the same the other way. The M-Pesa rows stay exactly as their messages have them.
3. **Each account code is a goal of the app** (a pocket), named and optionally locked until a day by the person.
4. **The balance is what the person types**, per goal when there are several, as one figure when there is one. Above the money paid in and taken out is interest (income, under Interest, never money arriving on Home); anything else is a correction. They are worked out again when older payments turn up.
5. **Asked once.** Under the first payment in Activity: "Is this your Chumz savings?" Add Chumz turns the older payments into transfers in one step and says how many changed; "It's not mine" stops the question and leaves the payments as they are.
6. **Where it shows.** A Savings apps card of its own in My accounts (the money is held outside M-Pesa; a line says it is paid in from M-Pesa), a page with the balance and Update balance, totals, goals and the money in and out, and on Goals "Saved in Chumz" with the lock date.

### The four additions the owner approved with the mockup

1. **First-balance interest rule.** The scan range may start after the first payment, so the first balance typed can hold money the app never saw. On a goal's first balance, the gap counts as interest only when it is at most 2% a month of the net paid in since the first payment found; otherwise it shows as "Was there before". There is no switch for it in this version; the page shows it on its own line so the lines add up.
2. **Remove Chumz, and add it again.** Chumz's page removes it, after asking: every payment counts as it did before and its goals, balances and interest go. Removing counts as "It's not mine", and either can be undone from My accounts › Add or remove accounts.
3. **Transfers into savings count toward the Plan's Savings line.** Money moved to any savings account (SACCO, money market, Chumz) is filed under Savings unless the person or the payee chose another category, counts as saved in the month and in the Health Index, and meets a planned Savings line. It stays an own transfer: never spending, and its other side is not income.
4. **Personal space only.** Chumz is offered and kept in Personal; other spaces are never asked (0016).

## Alternatives

- **Leave Chumz as a payee in the Savings category.** No schema work, and pay-ins count as saved, but money back stays income, there is no balance, interest or goal, and the person's net worth misses the money. Rejected.
- **Post M-Pesa straight against Chumz, with no mirror row.** One row per payment, but the journal walks each account's own rows, so a typed balance would be worked out without the pay-ins. Rejected for the mirror.
- **Put Chumz in the M-Pesa family (0026).** Shows it next to M-Shwari, but Safaricom does not hold the money and the family's rules (balance messages, one wallet at its head) do not fit an app with no messages. Rejected for a card of its own.
- **A balance checkpoint, as Set the balance does.** Reuses existing code, but one account-wide figure cannot say which goal the interest belongs to or keep goals apart. Rejected for typed balances per goal.

## Consequences

- Paying into Chumz no longer lowers spending or the Health Index; money back no longer reads as income.
- People without Chumz pay one small query per batch of new transactions.
- Mirror rows must follow their payments on every path that moves or deletes transactions (reading again, rechecks, copies, statement imports, moves); these paths are covered by tests. Payments into Chumz cannot be unmatched or moved to another space one at a time.
- The balance is only as fresh as the last one typed, and interest appears only then.
- A typo in an account code makes a stray goal the person can mark "not mine"; a code with a space in it is not read as a paybill.
- Nabo Capital's paybill may serve other products than Chumz; the question says "Is this your Chumz savings?" so the person can say no.
- Deferred: adopting a Chumz account made by hand, learned payee names, merging goals, business spaces, and a check on later updates that look like more than interest. The withdrawal message format is not confirmed; until a sample comes, money back that does not match stays as it is, and the person can still say it came from another of their accounts.
