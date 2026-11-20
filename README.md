# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.8.1
- **Native Version**: 2.8.1 (versionCode 281)
- **APK Size**: 47696264 bytes (~45.49 MB)
- **SHA-256**: `6b288b279e354b28595e9c5f5f6c9f4eef144ae6509df63906ca7de3aebc70b2`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Minimized Message Delivery, Doze Push Alerts & Faster Open-Room Sync**: Subscribed `customer_chats` to `"*"` (`INSERT` + `UPDATE`) while instantly dropping `_metadata`/`_threadSeenBy`/`_midIndex` echoes and skipping identical cached rows; added immediate Doze-safe Phoenix heartbeat probing, `IMPORTANCE_HIGH` background sound/vibration, 350ms debounced open-room `customer_chat_meta` catch-up with deterministic Page `"You: ..."` echo coalescing, and a 25s stale-socket heartbeat + single-delta guard.
  - **Hardened Chat Reply-To, Zero-Drain Reactions & Edits**: Preserved canonical `mid.$` in `sanitizeReplyMid()`, preserved reply fields on promotion/retry, fixed canonical MID matching and header offset for tap-to-scroll, propagated `message_edit` and `message_reaction` (including non-latest messages and `unreact`) directly into `ChatMessageLocalCache` + `_realtimeMessageEvents` (0 extra REST queries), and guarded `isEdited`/`originalText` against non-edited sync clobbering.
  - **Login-Epoch Auto End Shift & Cross-Device Roster/Lineup Sync**: Evaluated each rider's explicit Manila `loginDate`/`loginTimestamp` in Auto End Shift (protecting post-cutoff logins and unlogged zero-timestamp rows), added monotonic `lastUpdated` staleness guards in `applyRealtimeRosterUpdate()`, immediately cleared `customerName`/break timers/active catering cards on Available/End transitions, synced today's logins directly onto `_rosterList`, and added one-shot `RotationsScreen` on-enter / `ON_RESUME` ETag-cached checks.
