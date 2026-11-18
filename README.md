# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.6.1
- **Native Version**: 2.6.1 (versionCode 261)
- **APK Size**: 47371684 bytes (~45.18 MB)
- **SHA-256**: `a36392010f78b042e919339e837fa3da2e7ffd9202b164690814ebc865985a0a`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Silent Keep-Alive vs. High-Priority Alert Channels**: Separated persistent foreground service onto `lokalex_keep_alive_silent` (`IMPORTANCE_LOW`, zero sound/vibration/badge, `.setSilent(true)`, `.setOnlyAlertOnce(true)`) and routed genuine incoming messages/orders to `lokalex_chat_alerts_v2` (`IMPORTANCE_HIGH`) with `messageId` + persisted timestamp deduplication and clean Unmute handling.
  - **Swipe-Close, Screen-Off Doze & High-Priority FCM Delivery**: Added 3–4 minute Doze-safe `AlarmManager` watchdog & `onTaskRemoved` resurrection, timed `PARTIAL_WAKE_LOCK` on sync and realtime frames, fallback alert delivery inside `LokalexKeepAliveReceiver`, and high-priority data-only FCM wakeups (`priority: "high"`, `direct_boot_ok: true`).
  - **Rider-Scoped Cart Icon & Storage Reliability**: Strictly scoped Cart icons to each rider's own catered customers, unified `chat-media` bucket fallback probing (`chat-media` → `receipts` → `uploads` → `media`), and added direct compressed base64 JPEG Worker fallback for photos and receipts.
