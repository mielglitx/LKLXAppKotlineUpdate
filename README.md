# Lokalex Kotlin Native App Update Repository
Repository for base APK releases and app updates for Lokalex Kotlin Native.

- **Version**: 2.1.0
- **Native Version**: 2.1.0 (versionCode 210)
- **Release Highlights**:
  - **Native APK Installer Update**: Direct package installer prompt ensuring all native Kotlin background services, foreground notifications, and camera intents are updated.
  - **Locked Catering Card Selection**: Fixed active thread state binding (`Am(e)` and `PB()`) when selecting locked chat cards in the Catering tab so riders can inspect and unlock any card without selection bouncing.
  - **Customer Name Collision Fix**: Removed single first-name alias extraction in `Xf` and single-token matching in `Qf`, preventing customers with identical first names (e.g. Maria Dedios vs Maria Dela Cruz) from being incorrectly locked or attributed to the same rider.
  - **Null Notification Elimination**: Added strict null/blank/inbound validation in `LokalexForegroundService.kt` to prevent notifications with "null" content or title from being triggered by empty metadata updates.
  - **Native Camera Direct Capture**: Direct camera opening without file picker modal via FileProvider.
  - **High-Resolution Pipeline**: 2560px HD image processing across chat, canvas editor, and proof of delivery.
  - **24/7 Keep-Alive Service**: Persistent 20s background sync for roster and chats with screen off / app closed.
