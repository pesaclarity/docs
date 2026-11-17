# Statement import

How the app reads M-Pesa and bank statement PDFs on the phone. Part of [Getting data in](PRODUCT-SPEC.md#getting-data-in): SMS is the live source, and statements backfill history and check balances.

Status: proposed, 27 Oct 2026. Replaces the text-only M-Pesa reader with a layout-aware reader driven by statement packs, so banks can be added without app code.

## Principles

- **On the phone only.** The PDF, its password and its text never leave the phone. Only the fields a transaction needs are kept, plus a hash of each row for dedupe; row text is discarded after reading.
- **Packs, not code.** Each statement layout is a YAML pack in `ujumbe-parsers`, with synthetic fixtures, versioned and signed like the SMS packs.
- **Reconcile everything.** Every import is checked row by row (each balance follows from the last) and against the statement's closing balance; any gap is shown.
- **Synthetic data only** in repositories. Real statements are used to write and check packs on a contributor's own machine, and are never committed.

## Pipeline

| Step | Where | What it does |
| --- | --- | --- |
| 1. Open | App | Opens the PDF with PDFium; asks for the password if it is encrypted (see Passwords) |
| 2. Extract | App | Returns every word on every page with its position: page, left, top, right, bottom (PDF points, top-left origin). Runs off the UI isolate |
| 3. Detect | ujumbe | Picks the statement pack whose `detect` patterns match the first page |
| 4. Lay out | ujumbe | Groups words into lines, finds the header row, assigns words to columns, joins wrapped cells, and drops page headers, footers and repeated header rows |
| 5. Read rows | ujumbe | Turns each table row into a statement row: date and time, reference, description, money in, money out, balance, status |
| 6. Classify | ujumbe | Applies the pack's description rules to set transaction type, counterparty and account; folds separate fee rows into their payment |
| 7. Check | ujumbe | Balance continuity row by row; opening and closing balances; lines that looked like rows but could not be read |
| 8. Import | App | Dedupes against SMS (receipt, amount and time), adds the rest, records the import and shows the reconciliation |

Steps 3 to 7 are plain Dart with no Flutter dependency, so they run in CI against fixtures and in a phone isolate. The engine also accepts plain text for layouts that read correctly without positions.

## Layout

Text order alone loses table structure: columns merge, wrapped descriptions split rows, and amounts land in the wrong column. The layout step rebuilds the table from positions:

1. **Lines:** a word joins a line when their vertical spans overlap by at least half the shorter one, so small marks (commas, hyphens, asterisks) stay on their line; words are sorted left to right.
2. **Header row:** the first line on a page where the pack's column header patterns all match. Headers repeat on later pages and are skipped there.
3. **Columns:** the headers give a first guess, then each boundary moves to the gutter, the widest strip between the two headers' centres that no word of the table body crosses. M-Pesa statements centre their headers over the columns, so values start well left of their header or end well right of it; the gutters show where the columns really are. A word belongs to the column its centre falls in, except that only numbers go in amount columns.
4. **Rows:** a line starts a new row when the pack's `row_starts_with` cell has text and, if that column has a `pattern`, matches it (an M-Pesa receipt is ten letters and digits). Other lines continue the row above when they sit within two line heights of it, so wrapped descriptions join and page footers stay out.
5. **Chrome:** lines matching the pack's `skip` patterns (page numbers, disclaimers) are dropped, and so is anything above the header row.
6. **Order:** statements print rows from the same second in no reliable order (an overdraft draw, the payment it funds and its fee). Each such group is put in the order in which the balances follow.

## Statement packs

One YAML file per layout, in `packs/<id>/statement.yaml` (for example `packs/mpesa/statement.yaml`), with a JSON Schema in `schema/statement-pack.schema.json`. A sketch:

```yaml
schema: 1
id: mpesa_statement
institution: mpesa          # the SMS pack this statement belongs to
name: M-Pesa statement
version: 0.1.0
currency: KES
utc_offset: "+03:00"
detect:                     # any of these on the first page
  - '^\s*M-?PESA\s+STATEMENT\b'
period: 'Statement Period:? (?<start>\d{1,2} \w{3} \d{4}) - (?<end>\d{1,2} \w{3} \d{4})'
period_format: "d MMM yyyy"
columns:
  - { key: reference, header: '^Receipt No\.?$' }
  - { key: time, header: '^Completion Time$', format: "yyyy-MM-dd HH:mm:ss" }
  - { key: description, header: '^Details$' }
  - { key: status, header: '^Transaction Status$' }
  - { key: money_in, header: '^Paid In$' }
  - { key: money_out, header: '^Withdrawn$', negative: true }
  - { key: balance, header: '^Balance$' }
row_starts_with: reference
order: detect               # newest_first, oldest_first or detect
completed_status: '^Completed$'
skip:
  - '^Page \d+ of \d+'
  - '^Disclaimer\b'
fees:                       # charge rows folded into the payment they belong to
  description: '^[A-Za-z ]+ Charge$'
  link_by: reference
rules:                      # first match wins, on the description
  - id: paybill
    type: paybill
    pattern: '^Pay Bill(?: Online)? to \d+ - (?<counterparty>.+?) Acc\.? ?(?<account>\S.*)$'
  - id: sent
    type: sent
    pattern: '^Customer Transfer to (?<account>\S+) - (?<counterparty>.+)$'
password_hint: mpesa_statement   # key of an approved message in the app, not text
```

Amount styles a pack can describe: separate in and out columns (as above), one signed amount column (`amount`, with `negative_is: out`), or one amount column with a debit or credit marker (`amount` plus `marker: { column: dr_cr, out: '^DR$', in: '^CR$' }`).

**Fixtures** (`packs/<id>/statement_fixtures.yaml`, schema `schema/statement-fixtures.schema.json`): synthetic pages as positioned words, plus the rows and transactions they must produce. `tool/check_packs.dart` runs them, and a pack releases only when all pass, as for SMS packs. Fixtures are written by hand or generated from a real statement by a local tool that replaces every name, number, reference and amount before anything is saved.

**Updates:** statement packs ship in the same signed releases as SMS packs and are verified on the phone before use.

## Passwords

M-Pesa statements and most bank statements are encrypted PDFs.

- The app asks for the password when PDFium reports one is needed. It goes straight to PDFium, is not logged or stored, and the field is cleared once used.
- **Hints:** each pack names an approved hint message, for example where Safaricom sends the M-Pesa password. A locked PDF cannot be read before it opens, so the app shows the hints of the institutions ticked in My accounts (all of them when none is ticked). The app never guesses passwords.
- **Several files at once:** a person can pick several PDFs together. The first password that opens one is tried quietly on the rest of that batch, then dropped when the batch ends, is cancelled or the screen closes. A file it does not open asks for its own, and can be skipped.
- **Wrong passwords:** after three wrong tries in a row, a 30-second wait before the next.
- **Later, to confirm first:** if Safaricom's statement SMS carries the password, the app could offer to fill it in from that message, with the person's tap. Nothing is stored.

All new wording goes through the usual approval.

## Scanned and unusual PDFs

- **No text** (a scanned or photographed statement): the app says it cannot read this file and suggests downloading the statement again from the app or website. OCR is out of scope for now: ML Kit is a Google service the F-Droid build cannot use, and Tesseract is large.
- **Unknown layout:** the app names the institution if it recognised it and saves nothing. Contributors can add a pack.
- **Unread rows:** shown as a count with the reconciliation; the row text is kept only until the import screen closes.

## Performance

Measured on the reference phone in [performance-budget.md](https://github.com/pesaclarity/pesaclarity-app/blob/main/docs/performance-budget.md):

| Measure | Target |
| --- | --- |
| 12-month M-Pesa statement (about 100 pages), open to summary | 10 s or less |
| Peak memory during import | 200 MB PSS or less |
| UI frames during import | No frame over 32 ms; extraction and parsing run off the UI isolate |

## Build flavours

The F-Droid build has no PDFium ([0006](../decisions/0006-fdroid-build-without-prebuilt-libraries.md)), so it has no statement import. The engine steps (3 to 7) are pure Dart and ship in both builds, ready for a PDF engine that F-Droid can build from source.

## Rust

Decided in [0007](../decisions/0007-statement-engine-in-dart.md): the engine stays in Dart. Parsing a statement's text takes milliseconds, and the SMS and statement packs rely on Dart regular expressions (look-behind among them) that Rust's regex engine does not support. Rust is worth a time-boxed experiment for PDF extraction only, where a pure-Rust library could bring statement import to the F-Droid build. The experiment compares it with PDFium on synthetic statements for accuracy, speed, APK size and encrypted-PDF support.

## Rollout

1. **Engine and packs:** positioned words, layout, statement pack format and reader in `ujumbe-parsers`; the M-Pesa statement as the first pack, matching today's reader on the same fixtures.
2. **App:** the extractor returns positioned words off the UI isolate; the importer uses packs; today's text reader is removed.
3. **First bank:** KCB or Equity, from synthetic fixtures made with the local tool.
4. **Passwords and edge cases:** hints, batch imports, the wait after wrong tries, and the no-text message.
5. **Measure** on the reference phone, then run the Rust PDF experiment.
