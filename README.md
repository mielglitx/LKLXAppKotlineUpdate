# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.7.9
- **Native Version**: 2.7.9 (versionCode 279)
- **APK Size**: 47679860 bytes (~45.47 MB)
- **SHA-256**: `aa737c86f65ab77362ee733b1b7482f45114cf40001751468fc53007ab012c90`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Fixed Infinite "Loading messages..." Spinner on Empty Conversations**: Tracked per-room load completion (`loadedRoomIds`) with deduplicated `fetchRoomMessages`, clean `"No messages in this conversation yet"` empty-state placeholder, and zero-flicker loading guard so background delta syncs never flash spinners over rendered messages.
  - **Eliminated +1.21 MB Recomposition Room-Fetch Loop & Meta-Pulse Loops**: Keyed initial room hydration strictly on `canonicalCustomerId`, removed destructive preview-update full-room re-fetches, ignored null-message `customer_chats_meta` / `customer_chat_meta` pulses while WebSocket is connected, and short-circuited empty loaded rooms in `syncActiveThreadDeltaNow()`.
  - **Microsecond Watermarks, Zero-Polling Meta & 5s WebSocket Cooldown**: Preserved raw PostgreSQL 6-digit microsecond `updated_at` watermarks, applied `+1ms` boundary offset to `timestamp=gt.` queries, wrapped `refreshChatsDelta`/`refreshChats` inside `if (!isWsActive)`, added a 5-second WebSocket reconnect cooldown, and filtered read-receipt-only `riderLastSeen` echoes.
