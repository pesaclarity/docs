# 0026. M-Pesa as a family of accounts

Date: 2026-11-21 · Status: Proposed · Builds on [0018](0018-double-entry-journal.md), [0013](0013-review-only-accounts.md), [0016](0016-spaces-kept-apart.md) and [0009](0009-base-currency.md)

This is a plan. Nothing here is built. The direction was agreed with the owner; mockups go with the pull request and need the owner's approval before any screen is built. The open questions at the end need the owner's answers.

## Context

Most people's money in Kenya sits in M-Pesa and the products reached through it: Fuliza, M-Shwari savings and loans, Lock savings, KCB M-Pesa savings and loans, the Ziidi money market funds, and Pochi la Biashara. Pesa Clarity treats almost all of that as one wallet plus payees. What exists today (`pesaclarity-app` `origin/dev`, `ujumbe-parsers` `origin/main`):

- **One M-Pesa account.** `AccountType.mpesa` is a wallet with a reported balance ("New M-PESA balance"), one per line. `Institution.fromPack` maps the `mpesa` pack to it; everything else is a bank. The app's Cash account (`AccountType.cash`) is physical notes added by hand, and is not M-Pesa.
- **Fuliza is already a liability, but on the wallet.** `AccountOpening.canOweFuliza` is true only for M-Pesa. The journal keeps a `fulizaOwed` book account per M-Pesa account (`BookAccountKind.fulizaOwed`, liability) and `fulizaCharges` (expense). Fuliza messages carry `owed` and `due` (pack fields since engine 0.3.0), stored as `owedAfterMinor` and `owedDueAt`, a checkpoint on the debt as a reported balance is on the wallet. `FulizaAuto` works out draws and repayments the balances show without a message.
- **M-Shwari is a payee.** The `mpesa` pack reads moves to and from M-Shwari (`mshwari_deposit` as `paybill`, `mshwari_withdraw` as `received`). The M-Shwari balance those messages print is not read. `RuleCategoriser` files the payees `mshwari` and `ziidi` under the `savings` category. So a deposit is `LedgerRole.saving` and posts to the single fixed `BookAccount.savings`; a withdrawal is an inflow with a savings category, so `roleOf` makes it `ownTransfer`, posted to Money in transit with no other side to clear it. An M-Shwari balance message goes to the review inbox (fixture "M-Shwari balance goes to the review inbox").
- **Nothing else has rules.** No rule reads M-Shwari loans, Lock savings, KCB M-Pesa, Ziidi or a Pochi the person owns. Those messages go to the review inbox unless they happen to match a general shape (`received`, `sent`), where a loan draw could read as income and a repayment as spending. The statement pack (`statement.yaml`) has `unit_trust_in`, `unit_trust_out` and `small_business` (paying someone's Pochi, typed `till`).
- **Loans from messages.** The `kcb` pack's `loan_repayment` rule (engine 0.4.0) gives `owed` and `due`; `linkLoanRepayment` finds or makes the debt with that lender and records the balance left as confirmed. `LedgerRole.debtRepayment` posts the whole repayment to `BookAccount.borrowed`; only Fuliza has a fee share split out. The `Debts` table has no due date (only family loans have `DebtInstalments`).
- **Rules that span institutions.** A pack rule can set `institution: mpesa` (the `kcb` pack's `mpesa_paybill` and `mpesa_received`) so money is recorded on the M-Pesa account. Two messages about the same money on one account merge by `twinMatch` (`message_twins.dart`). Two sides of an own transfer pair through `own_transfers.dart` (`TransferPair`, fee allowance, 30-minute window, 3 days for one the person named) and meet in Money in transit (0018).
- **Goals and net worth.** `SavingsGoal.savedMinor` is typed by the person; no goal is backed by an account. `NetWorth.at` sums account balances (Fuliza taken off), money lent and debts. On hand leaves out savings accounts (`savingsAccountTypes`: `sacco`, `moneyMarket`).
- **Spaces.** `RoutingRules` routes a whole line (an institution id) to a business, or shares it with Personal (`LineChoice`). Nothing posts across spaces (0016).
- **Formats.** No public source has M-Shwari loan, Lock savings, KCB M-Pesa or Ziidi message wording. Ziidi is new, and Safaricom is changing how phone numbers are shown in messages.

The result: savings show as spending gone, withdrawals as money from nowhere, loans as income, and the person's real position is wrong in the way 0018 set out to stop.

## Decision

The M-Pesa balance is an **e-money wallet account**, like a bank current account. M-Pesa becomes a **family of linked accounts**, each its own ledger in the space's journal, with the wallet at its head. Money between the wallet and any member is an own transfer. Loans are debts with due dates; savings and funds are assets the person owns.

### 1. The members

| Member | Class | What its ledger holds |
| --- | --- | --- |
| M-Pesa wallet | asset | As today: every M-Pesa payment, the reported balance as a checkpoint |
| Fuliza | liability | As today (`fulizaOwed`, `fulizaCharges`), shown as a member rather than a hidden part of the wallet |
| M-Shwari savings | asset | Deposits, withdrawals, interest, balance readings |
| M-Shwari Lock savings | asset | Locked amounts, release at maturity, interest |
| M-Shwari loan | liability | Principal drawn, facility fee, a 30-day due date, repayments |
| KCB M-Pesa savings and target savings | asset | Deposits, withdrawals, interest, balance readings |
| KCB M-Pesa loan | liability | Principal, interest, due date, repayments |
| Ziidi money market fund, Ziidi Shariah | investment asset | Buys, redemptions, daily interest or returns, value |
| Pochi la Biashara (the person's own) | asset, business wallet | Sales received, money moved to the wallet; read in the Business space |

- **Storage.** Each member is a row in `Accounts` with `institutionId: mpesa` and a parent link to its wallet (a new nullable `familyOf` column), so two M-Pesa lines keep two families. A member's kind is a new field, not a new `AccountType` per product. Savings members count as savings (left out of On hand, as `savingsAccountTypes` are today); Ziidi is `moneyMarket`. Liability members are debts: each loan member has a row in `Debts` with its due date, and posts to its own liability book account instead of the shared `borrowed`.
- **Provider label.** KCB M-Pesa is in the M-Pesa family because it is opened, paid into and drawn through M-Pesa, and its messages concern the wallet. KCB shows as its provider label ("KCB M-Pesa · by KCB"). It is not linked to a KCB bank account.
- **Packs.** Rules for each member set the member as the account a message lands on, the way `institution: mpesa` does today (for example `account_kind: mshwari_savings`; the name is set in stage 2). Messages that only report a member's balance become **balance readings** for that member, not review items.

### 2. The honesty rules

These extend `roleOf` (`ledger_role.dart`) and the journal posting in `books.dart`:

- **Wallet to or from any member is an own transfer**, never spending or income. The wallet message and the member's own message (when there are two) are counted once: the same-receipt `twinMatch` on one account, then `TransferPair` across the two accounts, meeting in Money in transit. When only one message exists (the deposit message names both sides), it posts wallet against member directly.
- **A loan draw is borrowing**: the loan member goes up, the wallet goes up. Never income.
- **A repayment** takes the principal off the loan. The interest or facility fee part counts as a fee (`fees`), as the Fuliza fee share does today. When a message gives only the total, the split comes from the loan's own terms (fee or interest known at the draw); failing that, the whole goes to the loan and the next balance reading shows the rest.
- **Savings interest and fund returns are income**, labelled Interest or Returns, never Money received.
- **Paying someone else's Pochi** is spending, with that business as payee (the statement pack already types it `till`).
- **Money into the person's own Pochi** is business sales, read in the Business space.
- **Incoming money is told apart by source**: a person, a business or paybill, a bank, or a Pochi. Income is labelled from that, so a business payment is not "money from a person", and money from the person's own bank is an own transfer.
- **Balance readings are assertions** (0018): checked against the member's postings; a gap is flagged, never silently taken.

### 3. What the person sees

- **Accounts** groups M-Pesa with its members and their balances, wallet first, with Fuliza owed under it as today.
- **Debts** lists Fuliza, the M-Shwari loan and the KCB M-Pesa loan, each with what is owed and its due date, as in the approved KCB loan due-date design.
- **Goals** can be backed by an M-Shwari or KCB M-Pesa savings member (target savings included): what is saved is that member's balance, not a typed figure.
- **Net worth** counts savings and Ziidi as owned and the loans as owed. On hand and safe to spend stay the wallet and day-to-day accounts only.
- **Business.** A Pochi the person owns appears in the Business space with its sales; `RoutingRules` gains routing by member, not only by line.

### 4. Data

Message formats come from the owner's own recent messages, rewritten as synthetic fixtures (0013's rule: no guessed formats). There is no public source for most of these, Ziidi is new, and phone numbers are changing shape, so fixtures cover both the old and new number shapes where seen. Until a member's rules exist, its messages stay review items, as now.

### 5. Stages

| Stage | Work |
| --- | --- |
| 1 | This decision and mockups (Accounts, Debts, Goals, net worth, Business Pochi); owner's answers |
| 2 | Pack formats from the owner's samples: member rules and balance readings in `packs/mpesa`, fixtures, `docs/MESSAGES.md`; a new engine version if a new rule field is needed |
| 3 | App: member accounts and their book accounts, migration of existing M-Shwari and Ziidi rows, the honesty rules, Accounts, Debts, Goals and net worth screens, routing by member, the replay test extended to a month with M-Shwari, a loan and Ziidi |

## Alternatives

- **Keep them as payees and categories.** No schema work, but savings stay "spent", withdrawals land in Money in transit with nothing to clear them, loans read as income, and no balance can be checked. It is the problem 0018 exists to fix. Rejected.
- **One combined M-Pesa account with sub-balances.** Simpler screens, but an asset and a liability in one account do not net honestly, a move to M-Shwari would change nothing on the account, balance readings could not be checked per product, and goals and Debts could not point at a single member. Rejected.
- **Count M-Pesa as Cash.** M-Pesa is e-money held by Safaricom with reported balances; the Cash account is notes counted by hand. Mixing them loses the checkpoints. Rejected.
- **KCB M-Pesa under KCB.** Matches the lender, but its money moves only through M-Pesa, and most people with it have no KCB bank account. Rejected in favour of the M-Pesa family with KCB as provider label (open question 3).

## Consequences

- Balances, net worth and Debts reflect M-Pesa products as the person has them; moving money to savings no longer looks like spending.
- A migration moves existing M-Shwari and Ziidi rows to the new members and posts their past deposits and withdrawals as own transfers; anything it cannot place is reported, as 0018's migration does.
- The journal gains one book account per member and per loan instead of the shared `savings` and `borrowed` for these products.
- More accounts on screen: Accounts must group them so the wallet stays the obvious first line.
- Each new product format costs real samples from the owner before it is read; until then those messages are review items.
- KCB M-Pesa sits away from the KCB bank account; a person with both sees KCB in two places.

## Open questions

1. **Ziidi value.** No message gives a price or unit value. Is the value buys less redemptions plus returns messages, with any statement or balance message as a checkpoint?
2. **Lock savings maturity.** Show a maturity date, and treat the release as an own transfer back to M-Shwari or the wallet? What if it is broken early?
3. **KCB M-Pesa under KCB or M-Pesa.** Proposed: the M-Pesa family, with KCB as provider label.
4. **Interest accrual.** Post interest and returns only when a message reports them, or accrue daily (Ziidi) or monthly (M-Shwari) between messages?
5. **How a member is made.** Automatically on its first message, or offered to the person ("You have M-Shwari savings. Add it?")? Proposed: offered, with the first message's amount as its opening.
6. **Loan fee split without a message.** When a repayment message gives only the total, take the fee from the draw's terms, or leave it to the next balance reading?
7. **Pochi on a shared line.** When Personal and Business share the M-Pesa line, does the person's own Pochi always go to Business, even if no business space exists yet?
