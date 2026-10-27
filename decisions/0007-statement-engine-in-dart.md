# 0007. Statement engine in Dart

Date: 2026-10-27 · Status: Accepted

## Context

Statement import needs layout reconstruction and per-bank packs. Rust was suggested for speed. Parsing a statement's text takes milliseconds in Dart; the slow part is PDF text extraction, which is already native (PDFium). The SMS packs and the statement packs use Dart regular expressions, including look-behind, which Rust's `regex` crate does not support. Rust in the app would add a Rust and NDK toolchain, a Dart bridge, and a second language for contributors. A pure-Rust PDF library could, however, be built from source by F-Droid, which PDFium cannot.

## Decision

Keep the parser engine and packs in Dart. Run a time-boxed experiment on Rust for PDF extraction only, compared with PDFium on synthetic statements for accuracy, speed, APK size and encrypted-PDF support, and decide from the numbers.

## Consequences

- Packs stay one format, testable in plain Dart in CI.
- F-Droid statement import waits on the experiment.
- If the experiment wins, only the extraction step changes: it already hands the engine positioned words.
