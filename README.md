# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.1.5
- **Native Version**: 2.1.5 (versionCode 215)
- **APK Size**: 2380406 bytes (~2.27 MB)
- **SHA-256**: `1688ed192b3fec3315e27915ad7c417af53a41e3749cd082499646a05e107164`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Customer Done Status Revert Fix**: Prunes flushed roster mutations from the offline outbox, guards local roster state against stale snapshots, deduplicates dual-path metadata writes (`customerChatMeta/*`), awaits Done metadata confirmation, and blocks stale roster entries from pulling Done chats back into Catering.
  - **Mobile Data Usage Fix (System & Auth Configuration)**: Stops redundant resume/watchdog `goOffline()/goOnline()` reconnects and full non-volatile table re-syncs, adds row/room Postgres CDC filters (`room_id=eq.*`, `id=eq.*`) and exponential reconnect backoff (5s–60s), breaks the 118 KB `adminSettings/appErrorLogs` retry/broadcast loop, makes listener initialization idempotent with lazy Team Chat loading, and fixes data-usage bucket attribution.
