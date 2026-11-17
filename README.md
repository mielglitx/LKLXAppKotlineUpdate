# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.4.7
- **Native Version**: 2.4.7 (versionCode 247)
- **APK Size**: 47224972 bytes (~45.04 MB)
- **SHA-256**: `d7c61f365e3cfe9b887e9408ca675c3d1694d0737e0582c3d37e676f7ed89396`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Resilient Multi-Key Receipt Verification**: Enhanced `hasReceiptForCustomer` in `LokalexRepository.kt` to match by cleaned numeric PSID, candidate alias tokens (`|`, `/`), or normalized customer names. Removed fragile `minSessionTs` timestamp threshold cutoff and integrated `CartWizardDraftPrefs` active session checks.
  - **Reactive Concurrency & Inline Progress in Rotations**: Reactive `inFlightDoneMap` showing an inline `CircularProgressIndicator` on the `Done` button while executing within a coroutine scope, with dynamic receipt refresh fallback and clear warning toasts.
  - **Automated End Chat & Roster Transition to Available**: Automatically dispatches the closing auto-message to the customer thread, updates `customer_chat_meta` (`category = "Done"`), evicts the customer from `ActiveCateringLocalCache`, and seamlessly updates the rider's roster status back to `Available` (`isForcedCater = true`) when 0 active orders remain.
  - **Unified "• Done" Menu Action in Chat & Roster Sync**: Synchronized `canMarkDoneWithoutReceipt` (honoring Admin/TL overrides), `executeMarkCustomerDone` with `ActiveCateringLocalCache` eviction, and `releaseCustomerFromRosterCatering` with alias matching and `Available` status transition.
