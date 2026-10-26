# 0001. Flutter for the mobile app

Date: 2026-10-26 · Status: Accepted

## Context

The app needs a custom design that looks identical on cheap and expensive Android phones, heavy background parsing of thousands of messages, and an encrypted local database. iOS follows after the Android MVP. Options: Flutter, React Native, native Kotlin.

## Decision

Build the app in Flutter, Android first.

## Consequences

- Its own rendering engine keeps the design consistent across phones.
- Isolates run SMS and PDF parsing off the UI thread.
- Drift with SQLCipher is a mature encrypted database.
- The parser engine is pure Dart, so it also runs in CI without Flutter.
- Platform channels are needed for SMS access, Keystore and FLAG_SECURE.
