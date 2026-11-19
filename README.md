# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.7.3
- **Native Version**: 2.7.3 (versionCode 273)
- **APK Size**: 47601068 bytes (~45.40 MB)
- **SHA-256**: `70d39bea9c795bd63725be38550876ea89a68e50fbac6d6a8581866ceb6cb8b1`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **100% Manual App Update Checking with Zero Background/Startup Data Usage**: Eliminated all automatic background update checks, launch checks, and 30-minute background polling loops in `MainScreen.kt`. Startup checks local disk only (0 network bytes). Configured conditional HTTP `ETag` (`If-None-Match`) requests on Supabase `app_releases`, `settings?id=eq.app_releases`, and GitHub OTA `version.json`, returning `304 Not Modified` (0 bytes payload) on subsequent manual checks. Disabled background auto-downloads; APK files are downloaded exclusively when the rider taps **`Download & Install`**. Interactive Main Menu check with live visual feedback and toast notifications.
  - **Canonical Today's Gross Sync Across Rotations & Status Dock**: Extracted single shared helper `calculateTodayRiderGrossMap` in `LokalexRepository.kt` with strict `Asia/Manila` date scoping, voided/penalty exclusion, 5-minute receipt deduplication, and rider identity resolution, guaranteeing 100% exact parity between `RotationsScreen.kt`, the bottom status dock, and `CommissionScreen.kt`.
