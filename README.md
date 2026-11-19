# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.7.8
- **Native Version**: 2.7.8 (versionCode 278)
- **APK Size**: 47679860 bytes (~45.47 MB)
- **SHA-256**: `7afe4b6f8cb6993dd048f38a8cc7487452b82cbf8b721e49f772221e33784f7e`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Elimination of 48x/min Phoenix Realtime Loop**: Deduplicated PostgreSQL change channels with `ConcurrentHashMap<String, Boolean>` topic tracking, cutting 48 joins/min (33.5 KB) down to 1 on connect (0 KB steady state).
  - **Elimination of 39x/min Active Room REST Polling & -15s Rewind**: Removed negative rewind in `fetchMessagesDelta` and suppressed routine REST chat queries while Realtime WebSocket is connected, reducing traffic from 133.2 KB to < 1 KB.
  - **In-Memory WebSocket Updates for Customer Chat Meta**: Dispatched WebSocket metadata updates directly into memory (`applyRealtimeChatMetaUpdate`) without launching redundant REST queries; normalized cursor timestamps across Postgres `+00:00` and Kotlin `Z` formats.
  - **Bypassed 36.0 KB Background Catered Blob Fetch**: Gated heavy `/rest/v1/catered` downloads with smart hydration tracking, syncing receipts solely via lightweight delta watermark queries (< 2 KB).
  - **Accurate Caller Stack-Trace Unwrapping**: Filtered minified OkHttp interceptor frames (`l.intercept:15`) in `captureCallerTag()` to accurately identify true business logic repository callers.
