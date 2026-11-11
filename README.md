# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.1.4
- **Native Version**: 2.1.4 (versionCode 214)
- **APK Size**: 2377226 bytes (~2.27 MB)
- **SHA-256**: `ae0ab8ecb7c437943a9193e3f465f4dd5866af59744dd462e50902468f426839`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Base APK Updater Android Detection Fix**: Synchronously sets `window.isNativeKotlin` and `window.CapacitorCustomPlatform` in `index.html` before module initialization, updates `fC()`, and ensures `MainActivity.onPageFinished` preserves native flags so Android users never see `"Ang Base APK download ay para lamang sa Android app."`.
  - **10-Minute Background Crash Hardening (2 GB RAM)**: Handles `WebViewClient.onRenderProcessGone` safely, pauses background WebView timers/rendering in `onStop()` (keeping Picture-in-Picture active), removes offscreen `LAYER_TYPE_HARDWARE` FBO pinning, blocks background WebView IPC wakeups, and adds coroutine exception handlers plus Android 12+ exact-alarm guards.
  - **Store Menu Caching & Instant Repeat Sends**: Upgrades IndexedDB (`lokalex_idb_v2` v3) with `storeMenuGalleries` and `menuImagesCache`, caches gallery Blobs locally (120-image cap with ObjectURL cleanup), and preserves page `id`, `metaAttachmentId`, and `storeKey` across Quick-Send and Gallery flows.
  - **Live Chat & Update Cleanup**: Caps native/web chat list payloads, merges incoming messages incrementally, pauses hidden heartbeats/watchdogs, strips heavy `backdrop-blur*` layers, downscales upload photos off the UI thread, and removes legacy 0.x/1.x OTA clutter.
