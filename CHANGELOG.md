# Lokalex App Release Notes

## Version 2.1.6 (2026-11-12)

### 1. Background, Screen-Off & Resume Sync Resilience
* **Watchdog Delta Sync & Doze Recovery:** Fixed `REASON_WATCHDOG` skipping `syncDeltaNow()` in `LokalexForegroundService` and switched Doze alarm intervals automatically between 15 minutes (healthy WebSocket) and 3 minutes (disconnected/stale WebSocket).
* **Server-Clock Delta Cursor:** Separated PostgreSQL `updated_at` (`serverUpdatedAtMs`) from phone `lastUpdated` clocks so ahead-of-time client clocks cannot stall delta chat catch-up.
* **Guaranteed Resume UI Flush:** Preserved pending background `roster` and `customer_chat_meta` sync flags in `MainActivity.onResume()` and flushed missed updates into the WebView once timers resume.
* **FCM Push Payload Unwrapping (`send-fcm-push`):** Updated `supabase/functions/send-fcm-push/index.ts` to unwrap `record.data` for `roster` updates and emit collapsed silent sync wakes for non-unread chat folder/cater updates.

## Version 2.1.5 (2026-11-12)

### 1. Customer Done Status Revert Fix
* **Outbox Pruning & Write-Order Guard:** Marking a customer Done and returning to Available now clears queued roster mutations immediately after cloud confirmation and guards local roster state against stale in-flight snapshots so background syncs no longer flip the rider or chat card back to Catering.
* **Single Canonical Metadata Write & Done Lock:** Removed duplicate dual-path writes (`customerChats/*/metadata` vs `customerChatMeta/*`) and phantom ID variants during Done completion, awaited cloud metadata updates cleanly, and prevented stale roster entries from pulling Done chats back into Catering.

### 2. Mobile Data Usage Fix (System & Auth Configuration)
* **Eliminated Resync & Reconnect Storms:** Fixed background/foreground resume handlers, native `onResume`, and the sync watchdog from calling redundant `goOffline()/goOnline()` and full non-volatile table re-fetches (`adminSettings`, `teamChat/general`, `logins`, `settings/*`) every few seconds.
* **Targeted Realtime CDC Filters & Exponential Backoff:** Added row/room PostgREST CDC filters (`room_id=eq.*`, `id=eq.*`) so non-chat table listeners only receive matching events, and replaced fixed 3s channel reconnect loops with exponential backoff (5s–60s) gated on online/foreground state.
* **Stopped 118 KB Error-Log Sync Loop & Duplicate Listeners:** Fixed the broken `ResizeObserver`/network filter and added cooldowns in error reporting so transient offline errors no longer upload/re-broadcast `adminSettings/appErrorLogs`, removed eager admin error log realtime streaming, made `_F()`, `aO()`, and `lb()` idempotent, and made Team Chat general messages load lazily when opened.
* **Accurate Data Usage Attribution:** Classified `team_chat`, `orders`, `catered_history`, `customers`, `swap_requests`, and `map_calculations` accurately and excluded local WebView/blob fetches.

## Version 2.1.4 (2026-11-12)

### 1. Base APK Updater Android Detection Fix
* **Instant Native Detection Before Module Boot:** Set `window.isNativeKotlin` and `window.CapacitorCustomPlatform` synchronously in `index.html` and updated `fC()` + `MainActivity.onPageFinished` so Android APK users no longer hit the false `"Ang Base APK download ay para lamang sa Android app."` block.

### 2. 10-Minute Background Crash Hardening (2 GB RAM)
* **Safe WebView Renderer Recovery:** Implemented `WebViewClient.onRenderProcessGone()` in `MainActivity` to detach/recreate evicted WebView renderers without crashing the app or `LokalexForegroundService`.
* **Zero Background WebView Wakeups:** Paused `WebView` timers/rendering in `onStop()` (keeping Picture-in-Picture active), removed `View.LAYER_TYPE_HARDWARE` offscreen FBO memory pinning, blocked background JS IPC wakeups, and added `CoroutineExceptionHandler` + Android 12+ exact-alarm guards.

### 3. Store Menu Caching & Zero-Data Repeat Sends
* **Fixed IndexedDB Menu Object Stores:** Upgraded IndexedDB (`lokalex_idb_v2` v3) with missing `storeMenuGalleries` and `menuImagesCache` stores, reusing locally cached `Blob` thumbnails (`120`-image cap + clean ObjectURL revocation).
* **Instant Messenger CDN Reuse:** Preserved page `id`, `metaAttachmentId`, and resolved `storeKey` across Quick-Send and Gallery sends before modal teardown so repeat store sends reuse Meta `attachment_id` without re-downloading images.

## Version 2.1.3 (2026-11-12)

### 1. Zero-Polling FCM Push Wakeup Pipeline (Supabase-Only)
* **High-Priority Background Push Wakeups:** Added native Firebase Cloud Messaging handlers (`LokalexFirebaseMessagingService`, `FcmTokenManager`, `ChatAlertNotifier`) integrated with Supabase `fcm_tokens` and Edge Function `send-fcm-push` to wake Doze/closed devices on unread customer messages without continuous polling.
* **Delta Resume Sync & Legacy Cleanup:** Removed legacy Firebase RTDB HTTP fallbacks from `LokalexRepository` and the bundled app, switching reconnect/resume catch-up to lightweight Supabase `updated_at` delta queries (`refreshChatsDelta`) while suppressing notifications for outbound `You:` messages.

### 2. Android 12–15 Foreground Crash & Lifecycle Fixes
* **Foreground Promotion Safety:** Protected `LokalexForegroundService.startForegroundNotification()` with clean `stopSelf()` / `START_NOT_STICKY` handling and in-process `activeInstance` reuse across `LokalexKeepAliveReceiver` and FCM wakeups so background triggers never crash on Android 12–15.
* **WebView Readiness & Crash Diagnostics:** Guarded `MainActivity` WebView lifecycle callbacks with `isWebViewReady()`, added uncaught crash breadcrumbs (`last_crash.txt`), and exposed `NativePermissionsBridge.getNativePushDiagnostics()`.

### 3. Hardware Optimization (Snapdragon 450 · 2GB RAM · 64GB ROM · 4G · 4500mAh)
* **84% Smaller APK & 98% Smaller DEX:** Enabled R8 code and resource shrinking (`minifyEnabled true`, `shrinkResources true`) and removed 6 stale duplicate asset bundles, shrinking `app-release.apk` from ~14.8 MB down to **2.32 MB** (single ~856 KB `classes.dex`).
* **Battery & 4G Data Efficiency:** Removed permanent `FLAG_KEEP_SCREEN_ON` and deprecated `SCREEN_BRIGHT_WAKE_LOCK`, replaced infinite wake locks with bounded timeouts, tuned WebSocket/Phoenix keepalives to 45s with a 15-minute Doze-friendly watchdog, and lazy-loaded Tesseract OCR on demand.
* **Low-RAM Stability:** Turned off `offscreenPreRaster`, added RAM-only WebView cache trimming in `onTrimMemory` (preserving disk cache to avoid 4G re-downloads), and auto-cleaned old `camera_*.jpg` temp captures.

## Version 2.1.2 (2026-11-11)

### 1. Customer Locking & Notification Spam Fixes
* **Identical First-Name Isolation:** Fixed customer thread locking collision when multiple customers share the same first name.
* **Alert Deduplication:** Eliminated burst notification sound spam on rapid updates and bumped native build to `2.1.2`.

## Version 2.1.1 (2026-11-11)

### 1. WebRTC Camera Modal & Direct APK Installer Flow
* **In-App Camera Permission Bridge:** Added `WebChromeClient.onPermissionRequest` handling in `MainActivity` for WebRTC camera modals and enforced the direct APK installer update flow.

## Version 2.1.0 (2026-11-11)

### 1. WebView Asset Resolution & Native APK Updater
* **ServiceWorker Virtual Origin Fix:** Resolved virtual origin `https://appassets.androidplatform.net/` fetch handling in Android WebView, bumped `versionCode` to 210 / `versionName` to `2.1.0`, and configured pure APK installer updates (`isApkOnly: true`, `forceApk: true`).

