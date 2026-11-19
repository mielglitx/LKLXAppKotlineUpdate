# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.7.7
- **Native Version**: 2.7.7 (versionCode 277)
- **APK Size**: 47679860 bytes (~45.47 MB)
- **SHA-256**: `9c25862f58861d87090ba0d3455d7da7b3f63ed383ddf40b0cb79650c8a9e899`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **In-App 60-Second Network & Realtime Diagnostic Profiler**: Real-time network and WebSocket profiler with countdown timer pill, duration/hits/data summary bar, interactive Run/Stop trace controls, top callers & queries breakdown with hit counts, averages, query parameters chip, and structured Markdown report export to Android clipboard.
  - **Stack-Trace Call-Site Attribution & Query Logging**: Automatic stack-trace inspection via `captureCallerTag` isolating the exact business logic function and line number in `com.lokalex.rider.*`; full query parameter capture (`request.url.query`) and HTTP method tracking across all network requests and Phoenix Channels WebSocket frames.
  - **Root Cause Fix for Duplicate Realtime Byte Counting**: Fixed double counting in `sampleLiveTrafficStats` by managing a rolling credit balance against kernel `TrafficStats` and labeling residual OS-level socket traffic strictly as `Network & TLS Overhead` (never `Realtime (...)`); automated cleanup purging legacy duplicate records from disk storage; corrected OkHttp disk cache hit detection.
