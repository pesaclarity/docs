# 0034. Privacy-first web statement analyser

Date: 2026-11-25 · Status: Proposed · Builds on [0003](0003-parsers-as-a-separate-package.md), [0007](0007-statement-engine-in-dart.md) and [0020](0020-rayo-our-own-inference-engine.md)

Not agreed with the owner yet. A spike shows it can work; nothing is built for users.

## Context

Many people first want to see "where did my money go" from one statement, without installing an app. A web page can do that. Our promise is that financial data stays on the person's device, so the page must not upload the statement.

CountPesa offers a web analyser that processes statements on its server ("encrypted in transit and never stored"). Ours would never leave the device at all.

## Spike results

Branch `spike/wasm` in pesaclarity/rayo, commit `90177be`:

- rayo builds to WebAssembly with wasi-sdk 25.
- Size: generic 127 KB, SIMD 133 KB (about 60 KB gzipped).
- Outputs are bit-identical to the app goldens on 110 of 110 cases, in Node, celld and Chromium.
- Speed with the SIMD kernel: Ask reader about 1.4 ms per item, message reader about 0.6 ms per item.
- Memory: 16 MB.

## Decision

Build a web statement analyser that runs entirely in the browser.

- The user drops an M-Pesa or bank statement PDF on the page.
- PDF reading, parsing and the rayo models run in a Web Worker.
- Nothing is uploaded. The static site (pesaclarity.github.io) only serves files.
- Results show spending by category, top payees and fees, like the app.

### Work needed

- **Parsers and text encoders.** The statement parsers (ujumbe-parsers) and text encoders are in Dart. Either port them to JS or compile them to wasm. Either way, exact-match tests against the app goldens.
- **PDF passwords.** M-Pesa statements are password-protected. The password is asked for in the page and used only in the worker, never stored.
- **Fallback.** Load the generic wasm build when the browser has no wasm SIMD.
- **No third parties.** A strict Content Security Policy, no third-party scripts, no analytics on the page. The page works offline once loaded.
- **Lead-in to the app.** An "Export" button saves a file the app can import, so the person can continue in the app.
- **rayo.** The wasm SIMD kernel lands in rayo through a normal PR, with review and tests, not from the spike branch.

## Alternatives considered

1. **Server-side analysis (as CountPesa).** Simpler, but the statement leaves the device. Rejected: against our privacy promise.
2. **No web analyser.** Keeps the app as the only entry point, but loses people who will not install first.

## Risks and open questions

- Porting the Dart parsers: two copies can drift. Prefer compiling to wasm if the size is acceptable.
- Browser PDF text extraction may differ from the app's. Needs its own golden tests.
- Older or low-end phone browsers: memory and speed are untested outside Chromium.
- How to show users, in plain words, that nothing is uploaded (and let them check, e.g. it works with the network off).
- Bank statement formats beyond M-Pesa: which ones first.

## Consequences

- A free, private way to try Pesa Clarity, with a path into the app.
- rayo gains a supported wasm target that must stay bit-identical to the app.
- New code to keep in step with the app's parsers.
