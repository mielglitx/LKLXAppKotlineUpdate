# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.4.8
- **Native Version**: 2.4.8 (versionCode 248)
- **APK Size**: 47241356 bytes (~45.05 MB)
- **SHA-256**: `34ea72320297de6dbbd275d25d67135001e5cd38daeee73a1d3ec6520634da17`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Ungated Done Flow**: Removed the receipt verification gate from customer Done buttons across Rotations and Chat menus so orders can be completed immediately.
  - **Strict Receipt Lock on Void & Cancel with Admin/TL Override**: Prevents regular riders from voiding or cancelling orders once a receipt has been created, while allowing Admins and permitted Team Leaders to override and void the receipt and commission records.
  - **Instant Chat Done Optimistic Transition & 15s Grace Window**: Immediately updates thread metadata on frame 1 to Done, with a 15-second grace window to protect against background poll reversion, and auto-transitions the rider to Available when all orders are done.
  - **Direct OkHttp Streaming APK Downloader**: Replaces DownloadManager with direct OkHttp streaming on Dispatchers.IO, writing directly to downloads, throttling progress updates to 150ms, and auto-launching FileProvider package installer at 100%.
