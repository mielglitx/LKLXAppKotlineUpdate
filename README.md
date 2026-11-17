# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.4.6
- **Native Version**: 2.4.6 (versionCode 246)
- **APK Size**: 47224972 bytes (~45.04 MB)
- **SHA-256**: `9536eb78fa5a870324aeb6d741ae1407e37ea653c7c66fb3320cfce4ee884431`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Multi-Tier Chat Deduplication & In-Place Optimistic Reconciliation**: Canonical key pipeline (`client_msg_id` -> `metaMid` -> `id`), tight 15s content matching, in-place bubble replacement, and idempotent retry guards.
  - **Zero-Wait Frame 1 SQLite Load & Incremental Delta Sync**: Instant message emission from `ChatMessageLocalCache` on frame 1 without spinners, 60-message initial sync, and slim column projections (`select=id,room_id,sender_id,message,timestamp,data`).
  - **Adaptive Foreground Polling & Cursor Tracking**: Fast 2-second delta polling on active thread (with instant Realtime/FCM push trigger) + lightweight 6-second background delta on `customer_chat_meta`.
  - **Stable Compose Rendering**: Deterministic `key = { it.stableUniqueKey }` lambdas and `derivedStateOf` sorting on `filteredChats`, `displayChats`, and `displayMessages` to eliminate list recomposition flicker and scroll jumping.
