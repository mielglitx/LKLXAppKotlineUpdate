# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.7.6
- **Native Version**: 2.7.6 (versionCode 276)
- **APK Size**: 47663468 bytes (~45.46 MB)
- **SHA-256**: `fb308731a8e6f45cec39df6778424884861c44e47cfedc70b506f663c032c183`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Elimination of Roster & Logins Bandwidth Drain**: Realtime WebSocket in-memory updates for `roster` and `logins` (`INSERT`, `UPDATE`, `DELETE`) with local disk caching; suppression of all periodic polling ticks while WebSocket is connected; PostgREST delta watermark sync for `roster`; strictly today-scoped `logins` query eliminating the 50-row un-filtered historical fallback; targeted rider query in `updateRosterFields` eliminating 100-row table downloads; instant zero-byte in-memory status updates via `applyLocalRosterUpdate`; base64 `data:image` stripping on `Rider.photoUrl`.
  - **100% Dynamic Starting Hubs & Main Hub Resolution**: Dynamic derivation of starting hubs and branches from Supabase rates and settings without hardcoded town names; dynamic Main Hub configuration backed by `settings["mainHub"]` with direct Admin configuration modal access.
  - **Type-to-Suggest Cascading Location Dropdowns**: Material 3 `ExposedDropdownMenuBox` with dropdown arrows (`▼`), clear buttons (`✕`), and instant type-to-suggest filtering for Starting Hub, Destination Region, Destination Municipality/City, and Destination Barangay/District in Add/Edit Rate modals.
  - **Atomic Done Persistence & Farewell Auto-Message Guard**: Gated farewell messages strictly on active catering; optimistic in-memory Done transition with 15-second `recentlyCompletedGuard`; final Supabase `customer_chat_meta` PATCH ordered after farewell dispatch to prevent Done -> Inbox snapbacks.
