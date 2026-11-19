# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.7.4
- **Native Version**: 2.7.4 (versionCode 274)
- **APK Size**: 47617452 bytes (~45.41 MB)
- **SHA-256**: `26145ea7960f5993b08c3e44fc635d4376e8f636116a59ef0a2df97e3cf8e703`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Zero-Drain Delta Watermark Sync for Customer Chats & Chats Meta**: Cut 906 KB/min chat wire drain down to < 5 KB using persistent watermark queries (`timestamp > maxTimestamp`). Filtered Done threads on cold start and metadata sync (`data->>folder != done`), cutting metadata sync from 238 KB to < 3 KB, with on-demand reverse message pagination and lazy Done thread loading.
  - **Today-Only + Delta Watermark Sync for Receipts with Offline Cache**: Scoped routine receipt queries strictly to today's date in `Asia/Manila` with delta watermark tracking (`created_at > maxCreatedAt`), cutting receipt data usage from 191 KB to < 2 KB. Added instant offline hydration from `config_cache/receipts_today.json`.
  - **Realtime WebSocket Health Coordination & Polling Back-Off**: Exposed `isWebSocketHealthy` companion state so foreground REST loops back off to 20s–30s safety net intervals during active WebSocket streaming.
  - **100% Dynamic Directory Records**: Replaced all hardcoded town and store lists across the app with live Supabase records.
  - **Strict Catering-Gated Farewell Message Guard**: Gated `sendEndChatAutoMessageIfEnabled` in `markCustomerDone` strictly on active catering sessions, eliminating farewell auto-messages when archiving non-catering threads.
