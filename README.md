# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.1.6
- **Native Version**: 2.1.6 (versionCode 216)
- **APK Size**: 2380834 bytes (~2.27 MB)
- **SHA-256**: `44e4cc92d2f4e32e2846e6794a844dd8b4db01bae88921e24f704d7516e0e1a4`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Watchdog Delta Sync & Doze Recovery**: Fixes `REASON_WATCHDOG` skipping `syncDeltaNow()` and adapts Doze watchdog alarms between 15 minutes (connected) and 3 minutes (disconnected/stale WebSocket).
  - **Server-Clock Delta Cursor & Resume Catch-Up**: Uses PostgreSQL `updated_at` (`serverUpdatedAtMs`) to avoid client clock skew stalling deltas, and flushes pending `roster` / `customer_chat_meta` background events into the WebView on `onResume()`.
  - **FCM Push `roster.data` Fix**: Unwraps `record.data` for `roster` updates in `supabase/functions/send-fcm-push/index.ts` (requires Edge Function deploy) and sends collapsed silent sync wakes for non-unread chat updates.
