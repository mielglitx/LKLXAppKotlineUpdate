# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.4.9
- **Native Version**: 2.4.9 (versionCode 249)
- **APK Size**: 47241356 bytes (~45.05 MB)
- **SHA-256**: `7bef26830297f03de9d218c4f391d5ae5702d401856aed8c093126002af1c783`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Instant Chat Card Folder & Category Reactivity**: Resolved the issue where chat cards remained in active lists after status/folder changes (`Done`, `Cater`, `Undone`, `Followup` / `Scheduled`, `Applicant`, `Cancel`) until manual tab switches. Re-keyed `filteredChats` on Compose `mutableStateMapOf` snapshot state (`chatMetaMap`) and wired `updateChatMetaOptimistic` for instant frame-1 card relocation.
  - **Stable Item Identity (`psid`)**: Added `val psid: String get() = customerId.ifBlank { id.removePrefix("fb_") }` in `ChatMeta.kt` and wired `items(displayChats, key = { it.psid.ifBlank { it.stableUniqueKey } })` in `ChatThreadListPane.kt` for smooth exit transitions.
  - **Facebook Page Message Synchronization & Non-Text Attachment Fix**: Added global tail overlap polling (`fetchRecentGlobalMessages(limit = 80)` every 6s), preserved non-text and attachment-only messages (stickers, locations, media, links), auto-surfaced new and returning customers, and added 1-tap Force Message Update via Cloudflare Worker relay.
