# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.5.7
- **Native Version**: 2.5.7 (versionCode 257)
- **APK Size**: 47355260 bytes (~45.16 MB)
- **SHA-256**: `eeb3c70bbbc668e4fd61009b8627b3b0eb215d14924e4713890f7fedd7609277`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Outbound Sending Pipeline Fixes (Text, Gallery, Camera & Receipts)**: Normalized Worker `recipientId` (raw PSID) vs Supabase `room_id` (`fb_<psid>`), attached `apiId` and explicit `Content-Type: application/json`, sanitized `replyToMid`, synchronously cached picked/captured/receipt media into `cacheDir/outbound_media/` before dismissing modals, and added `CoroutineExceptionHandler` + `AppErrorTracker` + Toast error surfacing so failures are never silent.
  - **Single-Image Bubble Deduplication**: Deduplicated image URLs by base path & Facebook CDN keys and cleanly replaced optimistic local/Supabase URLs on confirmed server/echo reconciliation instead of concatenating duplicates.
  - **Rider Sender-Attribution Protection**: Preserved rider names (`sender = riderName`, `isFromPage = false`) across `send-reply`, SQLite cache, sync, and UI reconciliation so outbound rider messages never revert to `[Page]` / `Page Admin`.
  - **Insta-Replies Cleanup**: Removed hardcoded built-in templates (`Cater Greeting`, `Closing Message`, `On the way`, `Nandito na`, `Papunta na`) so only saved custom Supabase replies appear, all with Edit and Delete actions.
