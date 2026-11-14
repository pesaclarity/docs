# 0018. A double-entry journal is the source of truth for money

Date: 2026-11-14 · Status: Accepted

## Context

A tester's Personal space showed On hand KES 38,469 and safe to spend KES 4,946 when the real position was M-Pesa −1,300 (Fuliza owed), KCB 50 and cash 100. The app stores single-entry rows (one account, amount, in or out) and works out meaning and balances afterwards:

- An account's balance is the latest row's "balance after", plus later rows. Any such number wins, including one suggested by the message reader or typed while checking a message, and it is never checked against the previous balance and the rows in between.
- Messages still waiting to be checked count as nothing, yet Home shows confident figures.
- Fuliza and other debts are not accounts, so money owed is never taken off.
- Matching own transfers exists in code but never runs, so a transfer counts as spending on one side and income on the other.
- There is no account for cash or other money without messages, and no visible way to correct a balance.
- Business books are double-entry, but derived from the single rows, with any gap covered by a "balance difference" line.
- Tests used tidy synthetic data; none replayed a messy month against reported balances.

## Decision

Each space keeps a double-entry journal. Every money event is one journal entry with two or more postings, in integer minor units, that sum to zero per currency. Balances and every figure (On hand, safe to spend, Plan, Insights, reports, tax) are sums of postings.

- **Accounts**: assets (each wallet, bank, card, cash, sacco, money market), liabilities (Fuliza, loans, credit, money owed to suppliers), equity (opening balances, balance corrections), income and expense categories, plus two system accounts: **Not yet sorted** (suspense, for messages whose meaning is unknown) and **Money in transit** (clearing, for transfers whose other side has not arrived).
- **Messages and statements are source documents.** A parsed message posts its account against its category or system account. An unchecked message still posts its known amount against Not yet sorted, so balances stay right and Home can say how much is not yet sorted.
- **Reported balances are assertions, not truth.** Each reported balance is checked against the account's postings at that time. A mismatch is flagged with likely causes; it is resolved by fixing the cause or, with the person's confirmation, a visible balance correction posted to equity. A balance typed or suggested while checking a message is a claim and is only accepted once it reconciles or is confirmed.
- **Fuliza** draws credit a liability; repayments debit it; access fees are expenses.
- **Own transfers** go through Money in transit and clear when both sides match (fees allowed). Anything left in transit is visible.
- **Append-only.** Entries are never edited: changes are reversing entries plus new ones, which keeps the rule that transactions are never edited and gives a full history.
- **Spaces stay apart**: each space's journal lives in its own database; nothing posts across spaces.

## Consequences

- One migration rebuilds every space's journal from existing rows on upgrade, idempotently, and reports any account that does not reconcile.
- The derived business Books become views over the journal, with no plug lines.
- A permanent replay test feeds a messy synthetic month (unparsed messages, Fuliza, transfers, a wrong typed balance, duplicate import) and checks every account against its reported balances.
- Home shows uncertainty plainly ("KES 12,400 not yet sorted") rather than a confident wrong number.
- Prior art: GnuCash, Beancount and ledger-cli balance assertions, Firefly III.
