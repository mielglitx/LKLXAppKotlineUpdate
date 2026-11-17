# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.4.2
- **Native Version**: 2.4.2 (versionCode 242)
- **APK Size**: 47192204 bytes (~45.01 MB)
- **SHA-256**: `cc944903932e253de6b5fc6661507dd71ace427b200a9bbd25d7b61807416f00`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Full Jetpack Compose Native Upgrade**: Replaces legacy WebView shell (`v1.2.x` / `v2.1.x`) cleanly in place with matching signing certificate and `versionCode 242`.
  - **Scheduled Folder & Advance Order Multi-Stage Alarms**: Renamed `Follow Up` to `Scheduled` / `• + Schedule` (auto-pre-filling Advance Orders), added sorted Pending/Delivered/Cancelled sections with full rider actions, and added `30m / 15m / 5m / Due Now` audio + notification + snoozable banner alarms.
  - **Commission Rules, Cart Validation & Day-Off Highlights**: Enforced Commission priority order (`Promo Date -> Recurring Day -> Rider Override -> Configured Default`) with strict Admin editing, 3-condition Cart Wizard validation & auto-lock, and glowing stars-and-moon `🌙 DAY OFF` rider card borders.
