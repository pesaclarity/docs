# 0017. Ask: questions answered on the phone by two tiny models

Date: 2026-11-13 · Status: Proposed

## Context

People want to ask about their money in their own words ("how much did I spend on food last month?", "and only Naivas?") and get a short reply in plain language. A general language model (hundreds of MB) does not fit the 25 MB app, is slow on 2 GB and 32-bit phones, and can change or invent numbers. Federated learning on free text would risk leaking names, amounts and phone numbers. No data may leave the phone.

## Decision

Ask uses two small models in akili's TFLite pipeline, trained on synthetic data only, plus the app's own lookups:

- **Reader** (under 1M parameters, about 1 MB): picks the kind of question (spent, received, sent to, bills, debts, safe to spend, top items, small talk, and about 20 more). The details (category, payee, account, item, dates, amounts) are matched against the open space's own records with fuzzy matching and simple date rules.
- **Lookup**: the app runs the question on that space's database. Every figure comes from there.
- **Writer** (about 1 to 3M parameters, 2 to 4 MB): writes a short reply in plain language from the facts, using placeholders (`{amount}`, `{payee}`, `{month}`) that the app fills in. It never sees or writes numbers. A checker rejects any reply with a placeholder it was not given or a stray digit, and falls back to a fixed sentence. It also handles light small talk and "I can only answer from your records".

Ask stays inside the open space. Conversations: many per space, each titled from its first question, renamable, kept until deleted or for a period the person chooses (7 days to forever), with saving switchable off. Saved answers keep the numbers of their day and can be updated. Follow-ups change only what they name; the current what, who and when show as tags above the box. When the reader is unsure it asks. Answers to those questions become personal shortcuts ("chakula" means Food) for that space.

Ask has no federated learning, sends nothing anywhere, and its conversations are not in backups. English first; Swahili and Sheng later through more synthetic pairs.

## Consequences

- Replies are sometimes awkward and small talk is shallow; long open-ended chat is out of scope.
- Both models ship in the app and work on every supported phone, offline.
- New kinds of question are added by examples, a lookup and writer training pairs.
- The privacy policy gains a short Ask section: questions and conversations stay on the phone, per space.
