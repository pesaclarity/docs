# 0006. F-Droid build without prebuilt libraries

Date: 2026-10-27 · Status: Accepted

## Context

F-Droid rebuilds apps from source and accepts no prebuilt native libraries. The app used two: SQLCipher, downloaded by the `sqlite3` package's build hook, and PDFium, downloaded by `pdfrx_engine` for statement import. The spec and backups depend on SQLCipher. PDFium is large and hard to build from source.

## Decision

The `foss` build runs `tool/prepare_foss.sh` first. It removes PDFium, so the F-Droid build has no statement PDF import, and (once built) switches SQLCipher to compile from source. Our own `foss` release APK is built the same way, so F-Droid can verify it matches.

## Consequences

- F-Droid users get SMS capture and every other feature, but not statement import, until a PDF engine can be built from source (see the statement import spec).
- Two dependency sets to keep working; CI builds the prepared `foss` APK and fails if PDFium is in it.
