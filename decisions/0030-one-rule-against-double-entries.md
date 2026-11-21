# 0030. One rule that stops double entries for every account

Date: 2026-11-21 · Status: Accepted · Builds on [0018](0018-double-entry-journal.md), [0026](0026-mpesa-family-of-accounts.md) and [0028](0028-savings-apps.md)

Accepted by the owner on 21 November 2026. Built in `ujumbe-parsers` on `feat/own-transfer-codes` (the bank packs keep the M-Pesa code they print) and in `pesaclarity-app` on `feat/own-transfer-matching` (matching, the question and the repair), not yet merged.

## Context

Money moved between two of the person's own accounts sends two messages, one from each side, and sometimes a third: a bank's notice that money reached M-Pesa. Each message is read on its own, so one move can show as spending on one account and income on the other, or as the same money counted twice on one account. What exists today (`pesaclarity-app` `origin/dev` at 54219ec, `ujumbe-parsers` `origin/dev` at 14ded93):

- **One message makes one record.** `transactions_repository.addNew` never makes a second row for a message already stored (app#358).
- **Pairing by code, by number, or blind.** `own_transfers.dart` (`findOwnTransfers`) pairs two sides on different accounts that print one transaction code (`receiptNo` on both), pairs an outflow that names the account number it paid into, and otherwise pairs any send, paybill or bank outflow with any received or bank inflow of the same amount (allowing a fee) within 30 minutes. That last rule is a guess: a payment to a stranger and an unrelated deposit of the same amount minutes apart become an own transfer, and nothing tells the person.
- **The other side's code is dropped.** Airtel Money's received message prints the sender's code ("Sender TID"), Equity's send to a mobile wallet prints the M-Pesa ref, and KCB's "send to mobile wallet" prints "M-PESA REF". The packs read the first two into `account`, KCB's not at all, and the app keeps none of them, so these moves can only pair blindly.
- **One bank's notice.** The twin rule that merges a bank's notice on M-Pesa with M-Pesa's own message for the same money (`message_twins.dart`, `message_twins_store.dart`) knows only KCB (`walletNoticeBanks = {'kcb'}`, and M-Pesa naming "KCB").
- **A one-time repair.** `repairBalancesOnce` (`balances_repair.dart`, run from `pairHistoryOnceProvider`) merged stored notices and removed copies once. When the packs learn to read a code, history read before that is never matched again.
- **Own numbers.** The accounts list holds each account's number as its messages print it (`Accounts.reference`), the only record of which numbers and accounts are the person's own.

## Decision

One rule for every account: **a move between the person's own accounts is one transfer, never income or spending**, found in this order.

1. **One message makes one record.** Already built; it stays.
2. **One transaction code on two accounts is one move.** Two sides on different accounts, one out and one in, the same amount, that print the same code are one own transfer, whatever else they name. The code is the side's own receipt or the other side's code as the message prints it: Airtel's Sender TID, Etica's Transaction ref, Equity's and KCB's M-Pesa ref, a bank's reference. The bank packs capture the M-Pesa code where the message prints it: KCB's send to mobile wallet, and Loop, I&M, Equity and Co-op where their formats print it. The app keeps it on the transaction as the other side's code.
3. **With no shared code, one side must name one of the person's own accounts.** The two sides pair when one names an own account (an own phone number, an own masked account number, or an own bank's paybill with an own account number) and the other, on that account, has the same amount, allowing a fee difference, within about 30 minutes. A side that names another account of the person's, or a number that is not the other account's, or prints a code the other side does not have, is not that move.
4. **Own accounts come from the accounts list.** The app learns the person's own numbers and accounts from the numbers kept on their accounts; nothing is typed twice.
5. **When unsure, ask once.** A pair found by amount and time alone is a guess. It is held as a move (never counted as income or spending) and its line asks "Was this a move between your accounts?". The answer is remembered per sender (the party the money came from or went to as the message prints it): Yes keeps it and every later guess with that sender pairs without asking; No undoes it, both sides count as they were, and later guesses with that sender stay apart.
6. **Run matching again whenever the readers change.** `repairBalancesOnce` becomes a repair per reader version: when the engine or any bundled pack version changes, the notice merge, the copy removal and own-transfer matching run again over the whole history, then the journal is reposted.

A matched move is never income or spending, and each account's balance still follows its reported balances (0018).

**Notices from every bank.** The KCB-only twin rule becomes a table of the packs that send a notice when money reaches M-Pesa from an account with them, each with the name M-Pesa prints for that bank. KCB is the only entry today; a pack that gains a notice rule must gain an entry, and a test checks the table against the bundled packs.

## Alternatives

- **Keep blind pairing.** Pairs more moves with no work from the person, but some are a stranger's money, and a wrong pair hides income or spending without telling anyone. Rejected for the question.
- **Never pair without a code.** No wrong pairs, but most banks print no code the other side shares, so their moves would be spending and income again. Rejected.
- **Ask for every pair.** Honest but tiring: a person who moves money between two accounts weekly would answer weekly. Rejected for an answer remembered per sender.
- **A screen to list own phone numbers and accounts.** More complete than the accounts list, but it is a new screen and the same numbers are already on the accounts. Rejected for now.
- **Run the repair only once, as before.** Cheaper, but history read before a pack learned a code is never matched. Rejected.

## Consequences

- Moves between M-Pesa, Airtel Money, banks, Loop and Etica count once, with the fee still a fee.
- Some moves that paired before without a word now show the question once per sender.
- Transactions gain one column (the other side's code), filled when messages are read or read again.
- Matching runs over the whole history on the first start after each pack update; on a phone with years of messages that is a few seconds of work in the background.
- A pair the person answered No for stays apart even if a later pack reads a code that would pair it; they can still match it by hand.
- Formats without samples: Loop has no rules yet, and I&M's and Co-op's own-account sends to M-Pesa have no sample. Until the owner sends synthetic copies of those messages, their moves pair by own number or by the question.

## Owner's decisions

The six rules above are the owner's, approved on 21 November 2026 as "decision 0030: one rule that stops double entries for every account".
