# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.5.8
- **Native Version**: 2.5.8 (versionCode 258)
- **APK Size**: 47355268 bytes (~45.16 MB)
- **SHA-256**: `0ff4a7eb2ef0efae70c044739c40d6d597aaafebad86a135729b1fa8320f766c`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Fixed Active Chat Threads Jumping Back to Done**: Explicitly clears all completion fields (`completedAt`, `completedBy`, `completedByRiderId`, `completedByRiderName`, `lastMarkedDoneAt`, `is_done`, `isDone`) across both `customer_chat_meta` and `customer_chats` (`${custId}_metadata`) on every active/reopen transition (`Undone`, `Cater`, `Inbox`, `Scheduled`, `Applicant`, `Stores`, customer reply, or rider reply).
  - **Folder-Resolution Precedence & Grace-Window Hardening**: Enforces explicit non-done folder precedence over leftover `completedAt` or legacy `category = "Done"`, restricts local done-guard coercion to a 15-second post-Done grace window, fixes `ChatFeedScreen` local state traps so reopened threads update immediately, and resolves duplicate `<psid>` vs `fb_<psid>` alias rows in favor of newer active updates.
  - **Outbound Sending Pipeline & Media Reliability**: Preserves active `inbox`/`catering` state on outbound replies, normalizes Worker `recipientId` vs `fb_<psid>`, synchronously caches outbound media to `cacheDir/outbound_media/`, deduplicates single-image bubbles, and preserves rider sender attribution over `[Page]` echoes.
