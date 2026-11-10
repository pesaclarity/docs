# 0015. Gifts, not a paywall

Date: 2026-11-10 · Status: Accepted

## Context

Pesa Clarity has no ads and no investors, and money data stays on the phone. It still costs money: Apple charges US$99 a year to list an app, Google US$25 once, and there are the website, sign-in messages and the time it takes. An earlier idea was a paid licence for business features. Signal shows another way: everything free, one place to give, rare quiet reminders, and nothing unlocked by paying. CitaPay offers a tip-jar link that takes M-Pesa with no key in the app. Google Play and Apple expect tips to a developer to use their own billing, and an app that links out to pay may be rejected.

## Decision

Every feature stays free. People can give one-time gifts by M-Pesa through a giving page. In the app this exists only in the F-Droid build: a Support screen in Profile and a card on Personal's Home that asks at most once every 60 days, after the first 30 days, for 3 days at a time, never when money is tight and never within six months of a gift. Play and iPhone show nothing about giving; it is on the website's Sponsor page. The giving page, the payee names that identify a gift's M-Pesa message and the costs page are build settings (`tool/support.env` in pesaclarity-app); without a giving page nothing about giving shows.

## Consequences

- Nobody pays to use any part of the app, and a gift unlocks nothing. The paywall and licence idea is retired.
- The app sends nothing until the person taps Give; the browser opens the giving page with the amount only.
- A gift is recognised from its own M-Pesa message on the phone, so the app can thank the person and list their gifts without asking the payment service anything.
- Play and iPhone users give on the website, until the store rules are confirmed.
- Before the giving page goes live: the privacy policy (version 1.9) must say what the payment service and we receive (phone number, name, amount, M-Pesa code), and Citatech must confirm that tips are allowed under its terms. Whether gifts go to a personal account or a registered body is still open.
