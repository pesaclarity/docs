# 0013. Review-only accounts

Date: 2026-10-30 · Status: Accepted

## Context

People use institutions whose SMS wording is not public: no anonymisable sample of Loop's or
Airtel Money Kenya's own alerts could be found (Airtel Money Uganda and Tanzania samples use other
currencies and Swahili). Guessing patterns would misread money, and leaving the institutions out
of My accounts would leave their messages unread.

## Decision

- A parser pack may have no rules (`rules: []`). It still names its senders, so the app reads
  those messages, and every one goes to the review inbox, where the message reader (0012)
  suggests the fields.
- Loop (`NCBALOOP`, `NCBA_LOOP`) and Airtel Money (`AirtelMoney`) ship this way. Their sender IDs
  come from secondary sources and are confirmed against the first real messages.
- Rules are added once real wording is known, as synthetic fixtures, like any other pack.

## Consequences

- An institution can be offered before its wording is known; each new layout costs the person
  one check, and "Read messages like this the same way" covers repeats.
- A wrong sender ID means its messages are never read; nothing is misread.
- `docs/MESSAGES.md` in ujumbe-parsers lists every sample message, including these packs'.
