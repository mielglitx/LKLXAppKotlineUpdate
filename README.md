# Lokalex Kotlin Native App Update Repository
Repository for base APK installer releases and changelog for Lokalex Kotlin Native.

- **Version**: 2.1.3
- **Native Version**: 2.1.3 (versionCode 213)
- **Update Mode**: Pure Base APK Installer Only (isApkOnly: true, orceApk: true)
- **Release Highlights**:
  - **Zero-Polling FCM Push Pipeline (Supabase-Only)**: Added LokalexFirebaseMessagingService, FcmTokenManager, and ChatAlertNotifier for instant high-priority push notifications via Supabase cm_tokens + send-fcm-push Edge Function, and stripped legacy Firebase RTDB fallbacks in favor of updated_at delta catch-up sync.
  - **Android 12–15 Crash & Background Fixes**: Hardened LokalexForegroundService (startForegroundNotification() + stopSelf() fallback and in-process ctiveInstance reuse), guarded MainActivity WebView lifecycle callbacks, added last_crash.txt breadcrumbs, and exposed NativePermissionsBridge.getNativePushDiagnostics().
  - **Low-Spec Optimization (Snapdragon 450 · 2GB RAM · 64GB ROM · 4G · 4500mAh)**: Enabled R8 code/resource shrinking (~2.32 MB APK / ~856 KB classes.dex), pruned stale duplicate web bundles, lazy-loaded Tesseract OCR, disabled offscreenPreRaster, added RAM-only onTrimMemory clearing, removed permanent FLAG_KEEP_SCREEN_ON, bounded PARTIAL_WAKE_LOCK, and tuned 45s WebSocket heartbeats + 15m Doze-safe watchdog.
