# 0012. A message reader for unmatched messages

Date: 2026-10-30 · Status: Accepted

## Context

Parser rules only read the wording they were written for. Messages from a selected institution
that no rule matches waited in the review inbox with no screen, so new Saccos, banks and changed
alerts were lost to the ledger. Writing a rule per wording does not scale, and the person cannot
write one.

## Decision

- A small on-phone model, the message reader, tags each token of an unmatched message as amount,
  payee, date, time, balance, reference or fee, and the message as money out, money in or not a
  transaction. It trains on synthetic messages only, in pesaclarity-ml; the token pipeline is a
  shared spec with conformance vectors, as for the categoriser.
- Its output is only ever a suggestion: the person confirms or changes every value before a
  transaction is added.
- Confirming can save the message's layout ("Read messages like this the same way"), stored on
  the phone without amounts, names or dates. Later messages that fit it are added during capture.
- Until trained weights ship (pending the model licence), a rules reader makes the suggestions.

## Consequences

- New wording reaches the ledger after one check instead of a parser release.
- Evaluation holds whole sentence layouts out of training, which is the honest measure for new
  wording. Payee-first sentences the model has not seen are its weak spot.
- The reader needs no user data and no federated training, so it carries no privacy budget.
