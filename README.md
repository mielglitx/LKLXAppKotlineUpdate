# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.8.0
- **Native Version**: 2.8.0 (versionCode 280)
- **APK Size**: 47679880 bytes (~45.47 MB)
- **SHA-256**: `a58e1f2dbc94661269d21ef8b4dfdb3668f84629b9ee6773fac88623caa7b765`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Daily Asia/Manila Rollover & Date-Stamped Caches**: Centralized zero-network `ensureDailyRolloverIfNeeded` purges stale prior-day disk/memory caches on day rollover, stamps `cachedDate` inside `catered_today`, `receipts_today`, `roster_cache.json`, and `logins_today_cache.json`, and makes `todayDate` reactive across `MainScreen` and `CateredCustomersScreen`.
  - **Accurate Daily Tab Counters (`Catered`, `Logins`, `Rotations`)**: Scoped `cateredCount` strictly to today's non-voided, non-penalty receipts; treated empty HTTP 2xx `/rest/v1/logins` responses as authoritative to reset unlogged prior-day riders; and updated `RiderLoginsScreen` / `MainScreen` to count strictly via `hasLoggedInToday`.
  - **Missed-Cutoff Auto End Shift Recovery & Doze-Safe AlarmManager**: Replaced the 59-minute window with `lastDueCutoffMs` / `cutoffShiftDate` evaluation in `Asia/Manila` to catch missed overnight/Doze cutoffs (ending forgotten pre-cutoff `Catering` riders while protecting post-cutoff early logins), kept the 60s local timer at 0 bytes idle I/O, and added Doze-safe `ACTION_AUTO_END_SHIFT` `AlarmManager` scheduling.
