# 0014. Each amount at the rates of its day

Date: 2026-11-10 · Status: Accepted

## Context

[0009](0009-base-currency.md) converted every transaction at today's rate. Someone who counts in USD and pays from M-Pesa in KES saw last month's rent change value whenever the rate moved, saw KES payments only as small USD figures, and had amounts they typed in saved in USD even when they paid in KES. The free rates service (ExchangeRate-API's open endpoint) gives only the latest rates, not past ones.

## Decision

The phone keeps the first rates it fetches each day. A transaction converts at the rates of its day, or the closest earlier day's; one older than any kept rates uses the oldest, marked approximate. A transaction can carry the rate the person got, which wins. Lists show each amount in the currency it moved in first, with the base amount under it. A payment typed in takes its account's currency. Builds use the open endpoint by default (`tool/rates.env` in pesaclarity-app), with its credit line.

## Consequences

- Past totals stay put when rates move; only payments with no kept rates are approximate.
- The phone stores about 4 KB of rates a day, in the space's encrypted database. Nothing more leaves the phone than before: the same request, carrying only the base currency's code.
- Rates are per space, as the base currency is, so a space opened for the first time today has no rates for its earlier days.
- Past rates from before the app first fetched them would need a paid plan; the approximate label says so for now.
