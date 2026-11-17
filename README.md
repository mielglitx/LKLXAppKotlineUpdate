# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.4.5
- **Native Version**: 2.4.5 (versionCode 245)
- **APK Size**: 47224972 bytes (~45.04 MB)
- **SHA-256**: `61b24c75787e324bc60c9f9c6fc5aa1cd23894f63f7db212e8f3f27b47c0d97f`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Directory Customer PSID Sanitization & Real Name Restoration**: Detects numeric Messenger PSIDs and swaps actual customer names from subtitle, strips legacy `customers_` prefixes, purges unnamed numeric entries, and converts raw Maps URLs into clickable links.
  - **Add/Edit Record Form Layout & Inset Fix**: Partitioned system window insets (`setDecorFitsSystemWindows(false)` + bottom bar padding) so `Save Record` button is 100% visible and never pushed off-screen, and hid redundant FAB in form mode.
  - **Store Card "Send Menu" Horizontal Layout**: Enforced horizontal layout constraints on Store `Send Menu` buttons to prevent vertical letter-by-letter stacking.
  - **Commission Deduplication & Idempotency Locks**: Concurrency locks on "Done" and "Send Receipt", 10-minute pre-insert check, 5-minute receipt deduplication, and draft `receiptId` persistence.
