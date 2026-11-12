# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.1.8
- **Native Version**: 2.1.8 (versionCode 218)
- **APK Size**: 2382610 bytes (~2.27 MB)
- **SHA-256**: `8d41ef271694763ba00d9c8240b1c17d51f1aae9e3c946fb4bec42b6b6f64a88`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Receipt Wizard & Chat Back-Button Fix**: Prevents Android Back press from jumping back to `#view-wizard` or `#view-receipt-final` after sending a receipt directly to a customer or marking a customer Done.
  - **Modern UI & WebView History Guard**: Recognizes visible `#modern-ui-workspace` as `view-home` in `popstate` and back navigation, clears stale WebView back history via `NativePermissionsBridge.clearWebViewHistory()`, and routes hardware Back through `MainActivity.setupBackHandler()` + `window.handleNativeBackPress()`.
