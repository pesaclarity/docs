# 0011. The iPhone app

Date: 2026-10-29 · Status: Accepted

## Context

iOS lets no app read SMS, and the iPhone design should feel like iOS, not like the Android app. The Dart code (parsers, ledger, plan, insights) is the same on both. App Store rules and the lack of Google services on iPhone also shape it.

## Decision

- One Flutter codebase with an `ios/` project (bundle id `com.pesaclarity.app`, iOS 16+, iPhone only).
- Messages arrive through a Shortcuts automation that runs the app's "Add transaction from message" App Intent. The intent queues the message in a file protected until first unlock; the app reads the queue through the same channel names as Android's SMS inbox, so the scan, parser and review inbox are shared. Read messages are dropped; unread ones after 35 days.
- An adaptive layer switches widgets on the platform: system font, tab bar, grouped lists, alerts, pickers and segmented control on iPhone; Android keeps its design.
- The iPhone build is the `appstore` flavour: no Google services, so no Google sign-in, Drive or Play Integrity. Sign-in is email only until Sign in with Apple is added with its server endpoint.
- Linux CI cannot compile Swift; a manual macOS workflow builds the app unsigned.

## Consequences

- Capture needs a one-time manual automation; statement import covers anything missed.
- Queued message text sits on the phone, under iOS data protection, until read: the privacy policy must say so before release.
- The vault rejects iPhone backups until the server accepts App Attest; backups go to Files meanwhile.
- Release needs an Apple Developer account, signing and App Review notes explaining the automation.
