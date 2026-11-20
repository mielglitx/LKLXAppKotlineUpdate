# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.8.2
- **Native Version**: 2.8.2 (versionCode 282)
- **APK Size**: 47712648 bytes (~45.50 MB)
- **SHA-256**: `0e3359d526ce62b97ed7abe5257e6117b3ac310d9b64a04c6edf8ba4612858e8`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Cross-Device Lineup & Catering Timer Reconciliation**: Ensured `Available -> Catering` and `End` Realtime roster transitions are never blocked by slightly skewed or `0L` `lastUpdated` timestamps; aligned `ActiveCateredEntry.startedAt` and `RotationsScreen` `caterStartMs` consistently to `rider.lastUpdated` across devices.
  - **Empty `"Customer <ID>"` Ghost Card Removal**: Filtered out empty `"Customer <ID>"` (`No messages yet`) placeholder stubs across `Catering`, `Inbox`, and `Scheduled` (`followup`) in `ChatFeedScreen.filteredChats` and `ChatRepository.parseSupabaseChats()` with 0 extra network queries.
  - **Zero-Drain & Push Cleanups**: Removed 12 redundant post-`updateRiderStatus()` `refreshRoster()` calls in `RotationsScreen` and `MainScreen`, preserved microsecond watchdog cursor fallback via `ChatMessageLocalCache.getMaxChatTimestamp()`, and added `catered_by_rider_id`/`catered_by_rider_name`/`status` metadata to `send-fcm-push/index.ts`.
