# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.7.5
- **Native Version**: 2.7.5 (versionCode 275)
- **APK Size**: 47617452 bytes (~45.41 MB)
- **SHA-256**: `007ce209a0a240eb19f241b5e3da2c2fbb07ed020cb4d5158f0b5244601319bd`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Atomic Done Persistence & Elimination of Done -> Inbox Snapback**: Local `ChatMeta` immediately transitions to `Done` with 0 UI flicker, activating a 15-second `recentlyCompletedGuard[canonicalCustId]` before sending the End-Chat farewell message.
  - **Final Supabase Persistence Ordered After Farewell Dispatch**: `sendEndChatAutoMessageIfEnabled(...)` dispatches outbound farewell messages before sending the final `PATCH /rest/v1/customer_chat_meta` (`folder = "done"`, `status = "completed"`, `catered_by = null`), guaranteeing `folder = "done"` is authoritatively persisted in Supabase.
  - **Outgoing Message & Stale Delta Protection for Done Threads**: Outgoing page/rider messages and stale in-flight catering deltas are strictly blocked from reopening Done threads back to Inbox across all sync pathways (`parseSupabaseChats`, `maybeAutoReleaseDoneThread`, `fetchMessagesSb`, `fetchRecentGlobalMessages`, `forceMessageUpdate`, and `buildMetaUpdatesWithAutoRelease`).
  - **Strict Customer-Only Reopen Gate**: Only authentic inbound customer messages (`!isFromPage && !isOutgoing && !isRider`) with timestamps strictly newer than completion can move a thread back to Inbox.
