# Lokalex Kotlin Native App Update Repository
Repository for base APK releases and app updates for Lokalex Kotlin Native.

- **Version**: 2.1.1
- **Native Version**: 2.1.1 (versionCode 211)
- **Release Highlights**:
  - **WebRTC Camera Modal Permission**: Implemented `onPermissionRequest(request)` in `MainActivity.kt`'s `WebChromeClient`. Allows in-app camera viewfinder (`store-menu-camera-modal`) to stream live video directly without being blocked and falling back to the gallery camera folder.
  - **Native APK Installer Direct Update**: Streamlined the update prompt and download pipeline to directly download and launch the native Android Package Installer for `app-release.apk`, bypassing OTA zip updates and handling the "Install unknown apps" permission seamlessly on resume.
  - **Locked Catering Card Selection**: Fixed active thread state binding (`Am(e)` and `PB()`) when selecting locked chat cards in the Catering tab so riders can inspect and unlock any card without selection bouncing.
  - **Customer Name Collision Fix**: Removed single first-name alias extraction in `Xf` and single-token matching in `Qf`, preventing customers with identical first names (e.g. Maria Dedios vs Maria Dela Cruz) from being incorrectly locked or attributed to the same rider.
  - **Null Notification Elimination**: Added strict null/blank/inbound validation in `LokalexForegroundService.kt` to prevent notifications with "null" content or title from being triggered by empty metadata updates.
  - **24/7 Keep-Alive Service**: Persistent 20s background sync for roster and chats with screen off / app closed.
