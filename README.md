# Lokalex Kotlin Native App Update Repository
Repository for OTA updates and base APK releases for Lokalex Kotlin Native.

- **Version**: 2.0.1
- **Native Version**: 2.0.1
- **Release Highlights**:
  - **Camera Direct Capture**: Resolved issue where tapping camera opened the file picker. `isCaptureEnabled` now directly triggers the native camera via FileProvider.
  - **Ultra High-Quality Images**: Boosted photo, gallery, canvas editor, and POD resolution to 2560px/2048px with 0.85/0.92 quality (eliminating aggressive 720px / 0.45 compression).
  - **Screen-Off Background Sync**: Roster and chat synchronization now runs persistently on a 20s interval in background service with screen off / app closed.
  - **Roster Catering Lineup Fix**: Removed stale Firebase RTDB overwrites in catering confirmation and protected active Available rotation lineup so riders first in line are never falsely blocked with "You are on End Shift".
  - **Native Realtime WebSocket**: Phoenix Channels integration for sub-100ms instant customer chat push alerts.
  - **24/7 Keep-Alive Service**: Runs in dedicated `:sync` process with AlarmManager resurrection and battery optimization bypass.
