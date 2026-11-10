# 0009. Base currency, converted on read

Date: 2026-10-27 · Status: Accepted; converting at today's rate is replaced by the rates of each day in [0014](0014-rates-of-the-day.md)

## Context

The app assumed KES everywhere. People abroad, and people paid or charged in other currencies (a USD card subscription, a dollar account), need totals in the currency they count in, chosen at setup and changeable later, in any country. Converting means exchange rates, which change daily and come from outside the phone.

## Decision

The person picks a base currency at setup (their phone region's suggested) and can switch it in Profile. Transactions and accounts keep the currency they moved in and are converted into the base as they are read, at today's rate. Amounts the person types (plan, goals, debts, loans, subscriptions) are in the base currency; switching converts them once at today's rate after the person confirms. The phone fetches rates from a public service set at build time (`PESACLARITY_RATES_URL`), sending only the currency code, at most every 12 hours; rates the person sets take precedence. Without a rate, an entry keeps its own currency and stays out of base totals.

## Consequences

- The ledger is never rewritten for a rate, so switching back and forth does not drift transactions; typed amounts can drift by rounding on each switch.
- The rates service sees the phone's IP address and base currency; the privacy policy says so, and a person can use their own rates instead.
- Currency names, minor digits and region defaults come from Unicode CLDR, generated into the app, so no list is maintained by hand.
- Totals move when rates move, even with no new transactions.
