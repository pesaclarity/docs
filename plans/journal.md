# Plan: the double-entry journal (decision 0018)

Each stage ships on its own, with `tool/check` passing and the replay test growing.

1. **Journal underneath, nothing visible changes.** Tables `journal_entries` (id, occurred_at, source document, memo, reverses) and `postings` (entry, account, amount_minor signed, currency, rate). System accounts: Not yet sorted, Money in transit, Fuliza owed, Opening balances, Balance corrections, plus one per category. A posting engine turns each existing transaction row (and its role, fee, Fuliza split, transfer pair) into an entry; a migration rebuilds every space's journal, idempotently, and logs accounts that don't reconcile. Invariant test: every entry sums to zero per currency.
2. **Balances from postings, reported balances as checks.** Account balances, On hand and safe to spend read the journal. Each reported balance becomes an assertion with a pass/fail and a gap. Typed or suggested balances are claims until they reconcile or are confirmed. Unchecked messages post to Not yet sorted; Home shows how much is not yet sorted.
3. **Fixes as entries.** Set the balance, cash counts, corrections, unmatching and moving post reversing plus new entries; nothing is edited. Money in transit clears when transfers match, with fees.
4. **Everything else reads the journal.** Plan, Insights, Activity, reports, tax and the business Books become views over postings; the old "balance difference" plug goes.
5. **The replay test.** A synthetic month with unparsed messages, Fuliza draws and repayments, KCB → M-Pesa transfers with fees, a duplicate statement import and a wrongly typed balance; every account must match its reported balances, and On hand must match the sum the person would get by hand.

Out of scope here: moving money between spaces (still never), multi-currency own transfers (currencies stage 2).
