# Decisions

Architecture decision records (ADRs): one short file per decision that is expensive to reverse. Copy [0000-template.md](0000-template.md), take the next number, and open a pull request. Superseded records stay, marked with the record that replaced them.

| # | Decision | Status |
| --- | --- | --- |
| [0001](0001-flutter-for-the-app.md) | Flutter for the mobile app | Accepted |
| [0002](0002-licence-per-repository.md) | One repository per licence boundary | Accepted |
| [0003](0003-parsers-as-a-separate-package.md) | Parser packs as a separate Dart package | Accepted |
| [0004](0004-eu-hosting-and-email-sign-in.md) | EU hosting and email sign-in | Accepted |
| [0005](0005-pin-certificate-authorities.md) | Pin certificate authorities, not keys | Accepted |
| [0006](0006-fdroid-build-without-prebuilt-libraries.md) | F-Droid build without prebuilt libraries | Accepted |
| [0007](0007-statement-engine-in-dart.md) | Statement engine in Dart | Accepted |
| [0008](0008-dev-branch-and-release-please.md) | dev branch and release-please | Accepted |
| [0009](0009-base-currency.md) | Base currency, converted on read | Accepted; rates by day in 0014 |
| [0010](0010-dates-in-the-users-timezone.md) | Dates in the user's timezone | Accepted |
| [0011](0011-iphone-app.md) | The iPhone app | Accepted |
| [0012](0012-message-reader.md) | A message reader for unmatched messages | Accepted |
| [0013](0013-review-only-accounts.md) | Review-only accounts | Accepted |
| [0014](0014-rates-of-the-day.md) | Each amount at the rates of its day | Accepted |
| [0015](0015-gifts-not-a-paywall.md) | Gifts, not a paywall | Accepted |
| [0016](0016-spaces-kept-apart.md) | Spaces are kept apart, and people control them | Accepted |
| [0017](0017-ask-on-the-phone.md) | Ask: questions answered on the phone by two tiny models | Accepted |
| [0018](0018-double-entry-journal.md) | A double-entry journal is the source of truth for money | Accepted |
| [0019](0019-on-device-ml-without-google.md) | On-device ML without Google: one runtime for Play and F-Droid | Accepted |
| [0020](0020-rayo-our-own-inference-engine.md) | rayo: our own inference engine in C89 replaces TFLite | Accepted |
| [0021](0021-small-language-models-on-rayo.md) | Small language models on rayo, on the CPU | Proposed |
| [0023](0023-ml-on-pytorch.md) | pesaclarity-ml on PyTorch, without TensorFlow | Proposed |
