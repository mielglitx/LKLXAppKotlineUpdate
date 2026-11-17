# Lokalex App Release Notes

## Version 2.4.7 (2026-11-17)

### 1. Resilient Receipt Verification & Multi-Key Matching
* **Candidate Alias & Normalized Matching (`isCustomerNameMatch`):** Upgraded `hasReceiptForCustomer` in `LokalexRepository.kt` to match receipts by cleaned numeric PSID, candidate alias tokens (`|`, `/`), or normalized customer names (stripping legacy `customers_` prefixes).
* **Removed Fragile `minSessionTs` Threshold:** Eliminated the timestamp cutoff that previously rejected valid receipts generated earlier in the session or prior to server timestamp touch updates.
* **Active Cart Session Verification:** Integrated `CartWizardDraftPrefs` active session checks for the current day where `wizardStage >= 3`, `receiptLocked == true`, `receiptGeneratedAtMs > 0L`, `loggedToSupabase == true`, or a valid `receiptId` is saved.

### 2. Reactive Concurrency & Inline Progress in Rotations
* **Reactive Button Progress (`inFlightDoneMap`):** Replaced non-reactive concurrency sets with `remember { mutableStateMapOf<String, Boolean>() }` and rendered an inline `CircularProgressIndicator` on the customer row's `Done` button while the completion request is in flight.
* **Fuzzy Target Chat Resolution & Dynamic Refresh:** Resolved `targetChat` using candidate customer name matching and numeric ID equality, with a dynamic receipt refresh fallback from Supabase before displaying `"⚠️ Please generate a receipt for [Customer Name] before marking Done."`
* **Automated End Chat & Roster Transition to Available:** Automatically dispatches the closing auto-message to the customer thread, updates `customer_chat_meta` (`category = "Done"`), evicts the customer from `ActiveCateringLocalCache`, and seamlessly updates the rider's roster status back to `Available` (`isForcedCater = true`) when 0 active orders remain.

### 3. Unified "• Done" Menu Action in Chat & Roster Sync
* **Admin/TL Override & Cache Eviction:** Allowed `isTrueAdmin` and `hasEffectiveAdminAccess` overrides in `canMarkDoneWithoutReceipt`, added `ActiveCateringLocalCache` eviction to `executeMarkCustomerDone`, and upgraded `releaseCustomerFromRosterCatering` to use `isCustomerNameMatch` and auto-transition to `Available`.
* **Completed Order Filtering:** Upgraded `getUnreceiptedActiveCustomersForRider` in `RosterAndShiftRepository.kt` to ignore already-completed (`isDone` / `isCancelled`) orders.

### 4. UI Cleanups
* **Directory Dialog:** Corrected `+ + Add Location` button label to `+ Add Location`.

## Version 2.4.6 (2026-11-17)

### 1. Multi-Tier Chat Deduplication & In-Place Optimistic Reconciliation
* **Canonical Key Hierarchy (`stableUniqueKey`):** Introduced a multi-tier canonical key pipeline on chat messages prioritizing:
  1. Client Optimistic ID (`client_msg_id` / `effectiveClientId`): Attached to every outbound optimistic message and preserved across server confirmation echoes so the Compose key never mutates mid-flight.
  2. Messenger `mid` / Server ID: Primary key for customer inbound and cross-device rider messages.
  3. Deterministic Content Fallback: `${sender}_${timestamp}_${text.hashCode()}`.
* **In-Place Optimistic Bubble Replacement:** When confirmed server rows return via Supabase or Realtime, they match against the optimistic bubble by `client_msg_id` (with a tight 15-second fallback window for role/content) and replace the entry in-place rather than appending a second bubble.
* **Idempotent Retry Guard:** Message retries consistently reuse the original `client_msg_id` (`retryExistingId`), guaranteeing network retries never generate duplicate chat entries.
* **SQLite Cache Reconciliation:** Automatically deletes replaced temporary optimistic bubbles from `ChatMessageLocalCache` when confirmed server rows or Messenger `mid` entries arrive, and returns deduplicated messages on frame 1.

### 2. Zero-Wait Frame 1 Load & Incremental Delta Cursors
* **Instant Local-First Emission:** Immediately emits cached messages from `ChatMessageLocalCache` on frame 1 with zero spinner wait when opening a thread, followed by a background fetch of the latest 60 messages (`order=timestamp.desc&limit=60`) to backfill recent messages.
* **Slim Column Projections & Cursor Tracking:** Replaced unbounded message queries with explicit column selections (`select=id,room_id,sender_id,message,timestamp,data` on `customer_chats` and `select=id,data,updated_at` on `customer_chat_meta`).
* **Delta Sync by Timestamp & ID:** Tracks `threadLastTimestampCursors` and `threadLastMessageIdCursors`, querying only new messages with `timestamp=gt.$cursor` (and `id=gt.$afterId`) without re-downloading older history.

### 3. Adaptive Foreground Polling & Stable Compose Rendering
* **Fast 2-Second Active Thread Polling:** Adaptive polling queries deltas every 2 seconds for the open active conversation (with instant wakeup on Supabase Realtime events and FCM push notifications).
* **Lightweight 6-Second Background Metadata Polling:** Added an independent 6-second delta sync on `customer_chat_meta` for lightweight folder and unread badge updates without wasting mobile bandwidth.
* **Jank-Free LazyColumn Rendering:** Applied deterministic `key = { it.stableUniqueKey }` lambdas and `derivedStateOf` sorting on `filteredChats`, `displayChats`, and `displayMessages`, preventing list recompositions, layout flicker, and scroll jumping.

## Version 2.4.5 (2026-11-17)

### 1. Directory Customer PSID Sanitization & Real Name Restoration
* **Automatic PSID & Name Inversion Detection:** Fixed customer directory cards displaying raw numeric Facebook Messenger PSIDs (e.g. `26058989460442650`, `26155948474078057`, `26610832338612974`) under `# SECTION` by detecting when the stored `title` is numeric while the customer's actual human name is stored in `subtitle` (e.g., `Bavelyn Abacan Mariano`, `Len Valdez`, `Jennifer Galicia`). The parser now automatically swaps the real customer name into the primary title and preserves address/landmarks in the subtitle.
* **Legacy Prefix Stripping:** Strips legacy webhook prefixes such as `customers_` and `customer_` along with pipe delimiters (e.g., `"customers_asia tangonan | asia presentacion"` -> `"Asia Tangonan"`).
* **Purge Phantom Records & Map Link Normalization:** Drops raw numeric entries with no human customer name from the customer directory and automatically parses raw Google Maps URLs located in `subtitle` or `detail` into clickable `📍 Open in Google Maps` links with default `"Camiling, Tarlac"` location text.

### 2. Add/Edit Record Form Layout & Navigation Bar Inset Fix
* **Pinned Action Bar Window Insets:** Configured `WindowCompat.setDecorFitsSystemWindows(dialogWindow, false)` and partitioned `navigationBarsPadding().imePadding()` exclusively on the pinned bottom action bar `Surface` inside `DirectoryRecordFormView`. This completely fixes the issue where the `[Cancel]` and `[✓ Save Record]` buttons were pushed 95% off the bottom edge of the screen on gesture navigation devices.
* **Contextual FAB Visibility:** Automatically hides the floating `+` FAB while actively inside the Add/Edit Store or Customer form.

### 3. Store Card "Send Menu" Button Horizontal Flex Fix
* **Fixed Button Orientation:** Enforced `defaultMinSize(minWidth = 76.dp)` and `maxLines = 1` with `softWrap = false` on the orange `Send Menu` button on Store cards, completely eliminating the vertical letter-by-letter collapse (as observed on `Aling Inday`).

### 4. Commission Record Idempotency Locks & 5-Minute Deduplication
* **In-Flight Concurrency Locks:** Implemented `inFlightReceiptLocks` in `saveReceiptToSupabaseDetailed` and concurrency sets in `RotationsScreen.kt` to prevent duplicate submissions from rapid double-tapping on "Done" or "Send Receipt".
* **10-Minute Pre-Insert Duplicate Check:** Before inserting a delivery receipt to Supabase, queries existing records for the same rider, customer, date, and fee amount; skips redundant inserts if an identical record was saved within 10 minutes.
* **Client-Side Deduplication (`deduplicateReceipts`):** Merges duplicate receipts within a 5-minute window matching rider, customer, date, and gross fee in `CommissionScreen.kt`, while strictly preserving legitimate penalty adjustments.
* **Draft `receiptId` Persistence:** Retains and updates the canonical `receiptId` in `CartWizardDraftPrefs` across draft loads and saves to prevent duplicate receipt rows when re-editing or resending an active customer order.

## Version 2.4.4 (2026-11-17)

### 1. Admin & Permitted Team Leader (TL) Force-Status Controls
* **2x2 Force-Status Actions Grid in Rotations:** Inside the rotation rider detail modal (`RiderQueueDetailsDialog`), Admin and permitted Team Leaders now have a 2x2 grid of action buttons:
  * **`🟢 Available`**: Shows confirmation prompt, resets the rider's available queue timestamp, clears active orders, and updates status to `Available`.
  * **`🛵 Cater`**: Directly opens `CaterCustomerDialog` with customer input and recent active inbox chat picker, setting status to `Catering`.
  * **`☕ Break`**: Directly opens `RiderBreakDialog` to choose break durations (15m, 30m, 45m, 60m, Unlimited), setting status to `Break`.
  * **`🛑 End Shift`**: Shows a high-contrast confirmation alert, clocks the rider out, and terminates duty (`status = "End"`).
* **Team Leader Permission Toggle & Gate:** Added `canForceStatus` to `TL_PERMISSIONS_LIST` with a dedicated toggle in Admin TL Settings (`Allow TL to Force Rider Status (Available / Cater / Break / End)`) backed by `canForceRiderStatus(currentUser)`. Unauthorized riders and TLs without permission retain a clean read-only inspection modal.
* **Immediate Lineup Refresh & Receipt Gate Bypass:** Manual Admin/TL status overrides pass `isForcedCater = true` and `forcedBy` to bypass self-serve receipt guards, trigger immediate lineup refreshes via `refreshRoster()`, and provide instant feedback toasts.

### 2. Rotation Receipt Validation Gates & Shift Reversion
* **Receipt Gate on Customer "Done":** When tapping "Done" on a customer in the Catering section of Rotations, the app checks if an official receipt has been generated. If none exists, completion is blocked with a warning: `⚠️ Please generate a receipt for [Customer] before marking Done.`
* **Auto-Revert to Available:** Once all active deliveries for a rider are marked Done, the rider's roster status automatically reverts to `Available` with a refreshed queue timestamp.
* **Receipt Gate on Status Dock "Available":** When a rider taps `Available` on the status dock while holding unreceipted active orders, the transition is prevented until receipts are created or orders completed.

### 3. Full Directory Sync Pagination & UI Layout Fixes
* **Paginated Full Directory Sync:** Removed the PostgREST 1,000-row limit in `LokalexRepository.kt` by paginating in chunks with Range headers until all records are synced, ensuring stores (e.g., Mayummy) and customers (e.g., Kurt Gabres) are never cut off.
* **Customer Data Cleanup:** Automatically filters out raw numeric-only Messenger PSID entries from top-level customer cards and strips legacy `customers_` prefixes from customer names.
* **Store Card "Send Menu" Button Layout:** Corrected row flex/weight wrapping so the orange "Send Menu" button remains horizontally oriented across all screen densities.
* **Leaflet Map Picker & Address Modal:** Fixed map picker coordinates extraction, full address display, and GPS auto-fill in `AddAddressModal` and `DirectoryDialog`.

### 4. Real-Time In-App APK Download Progress Bar
* **Chunked Streaming Downloader:** Implemented `ApkUpdateDownloader.kt` streaming updates via OkHttp in 32KB chunks with 100ms UI throttled state emissions.
* **Smooth Visual Progress UI:** Displays an animated `LinearProgressIndicator` in the Update Dialog and top status banner, showing real-time percentage (`48%`) and downloaded size (`14.2 MB / 28.5 MB`) with indeterminate fallback if Content-Length is missing.

## Version 2.4.3 (2026-11-17)

### 1. Customer Seen & Rider Read Indicators
* **Customer Seen Badge (`✓✓ Seen`):** Outbound messages now display a vivid sky-cyan `✓✓ Seen` indicator once read by the customer (via Messenger read watermarks, customer read timestamps, or subsequent customer inbound replies).
* **Rider Avatar & Initial Badges:** Outbound and customer messages render compact amber circular initial badges alongside a `"Seen by [Rider Name(s)] • [Time]"` summary underneath the specific message each rider in the shift has last read, synced across riders via Supabase Realtime (`_threadSeenBy`).

### 2. Edited Message History Sub-Label
* **Preserved Initial Text & Sub-Label:** Extended message webhook and Supabase message decoders to track `original_message`, `originalMessage`, `original_text`, `originalText`, and `edit_history`.
* **Tap-To-Expand Original Text:** Underneath any edited bubble, renders a subtle, muted sub-label showing `✏️ Edited • Original: "[original message text]"` with tap-to-expand support for long edits.

### 3. Compact & Scannable Insta-Reply Cards
* **Redesigned Quick-Reply Layout:** Restructured template cards into a streamlined two-line format featuring the bold template title on line 1 and a single-line ellipsized preview (`maxLines = 1, overflow = TextOverflow.Ellipsis`) on line 2 with tight vertical padding.
* **Component Extraction (`CompactQuickReplyItemCard`):** Modularized reusable compact item cards in `ChatRoomComponents.kt` and `LokalexSubMenuDialogs.kt` (`QuickRepliesDialog`), allowing more templates to fit on-screen while preserving full tap-to-insert and 1-tap send capabilities.

## Version 2.4.2 (2026-11-17)

### 1. Full Jetpack Compose Native Release & Safe In-Place Upgrade (Build 242)
* **Unified Package Upgrade Compatibility:** Bumped native release metadata to `v2.4.2` (`versionCode 242`) with matching signing certificate (`com.lokalex.rider`), allowing seamless in-place APK installation over both Android WebView (`v1.2.x` / `v2.1.x`) and earlier Native builds without `INSTALL_FAILED_VERSION_DOWNGRADE` or signature mismatch.
* **Live Package Version & Direct APK Downloader:** Updated `LokalexMenuDialog`, `ReleaseNotesDialog`, and `AppUpdateDialog` to display the live `PackageManager` version/build code, perform safe manual `version.json` checks without false downgrade prompts, and download/install verified `.apk` packages directly via `MainActivity.installDownloadedApk` and `FileProvider`.

### 2. Scheduled Folder, Advance Order Sections & Multi-Stage Alarms
* **Scheduled Category & `+ Schedule` Action:** Renamed the `Follow Up` category tab and menu action to `Scheduled` / `• + Schedule` (preserving backward compatibility with legacy `"followup"` / `"follow_up"` / `"Scheduled"` metadata tags). Tapping `• + Schedule` tags the thread and launches the Advance Orders modal pre-filled with the active customer's name.
* **Sorted Collapsible Sections & Rider Controls:** Reorganized Advance Orders into Pending (scheduled ascending), Delivered (delivered descending), and Cancelled groups with inline Claim, Assign Rider, Mark Done, Mark Cancel, and Edit Order (DatePicker, TimePicker, Notes) actions.
* **Multi-Stage Alarms & Interactive Top Banner:** Added background/foreground alarm evaluations (`30 mins before`, `15 mins before`, `5 mins before`, `On Scheduled Time`) with milestone deduplication, 5-minute Snooze, dual-tone audio alerts, system notifications, and a tap-to-scroll pulsing highlight inside `AdvanceOrdersDialog`.

### 3. Commission Rule Resolution Priority & Strict Admin Editing
* **Strict Rate Priority Order:** Removed hardcoded `15%` regular fallback so commission resolution strictly evaluates **Specific Promo Date Rule -> Recurring Day Rule -> Individual Rider Override (Rider ID / normalized Name) -> Configured Default Rate**, applied consistently across rider cards, company due totals, net take-home, and generated text reports (including Admin 0% exemption).
* **Commission Settings & Role Locks:** Added interactive Promo DatePicker, numeric inputs, and full-field saving inside `AdminCommissionSettingsDialog`, and restricted manual commission records, date penalties, receipt edits, and rule saves strictly to `isTrueAdmin`.

### 4. Cart & Receipt Wizard Validation, Locking & Receipt Improvements
* **Strict 3-Condition Item Validation & Market/Handling Exclusivity:** Enforced item completeness checks (`On Hand`, `Market/Store`, `Price > 0 or Paid`) before receipt generation, restricted discounts to fees only, and applied Market Fee vs. Handling Fee exclusivity when carts are 100% Market or 100% Store.
* **Auto-Lock & Dimming Overlays:** Automatically locks the cart upon receipt generation, send, or Done, displays a dimmed overlay with change calculator and unlock confirmation on Steps 1 & 2, downscales/compresses direct receipt JPEG images for fast sending, and renders a larger top header logo.

### 5. Day-Off Stars & Moon Highlights, Modularity & Chat Reliability
* **Scheduled Day-Off Today Highlight:** Riders logged in or active on their scheduled day off (`Asia/Manila`) display a glowing stars-and-moon border (`Modifier.dayOffStarsAndMoonBorder`) and `🌙 DAY OFF` pill across Login and Rotation cards.
* **Inbox/Done Folder Guards, Chimes & Modular Repositories:** Hardened Inbox/Done folder transitions and realtime message deduplication, improved `RingtoneManager` chime playback, added `HorizontalPager` image viewer with edit/document download support, and modularized repositories (`ChatRepository`, `MediaUploadRepository`, `RosterAndShiftRepository`) and chat UI panes.

## Version 2.1.8 (2026-11-12)

### 1. Receipt Wizard & Chat Back-Button Navigation Fix
* **Fixed Wizard Re-Entry on Back Press:** Resolved an issue where pressing the Back button on Chat after sending a receipt directly to a customer—or after marking a customer Done—would jump back into `#view-wizard` / `#view-receipt-final` instead of staying on Chat/Home.
* **Modern UI & Android WebView Back Stack Guard:** Updated `_lokalexActiveViewId()` and `popstate` so Modern UI (`#modern-ui-workspace`) is properly recognized as `view-home`, cleared stale wizard history on receipt completion (`ZF()` / `completeReceiptDone`) and Done completion (`Y_()`) via `NativePermissionsBridge.clearWebViewHistory()`, and wired `MainActivity.setupBackHandler()` through `window.handleNativeBackPress()`.

## Version 2.1.7 (2026-11-12)

### 1. Gallery Multi-Selection Upload Fix
* **Android WebView Multi-Select Intent:** Updated `MainActivity.onShowFileChooser()` to pass `Intent.EXTRA_ALLOW_MULTIPLE = true` when `FileChooserParams.MODE_OPEN_MULTIPLE` is requested and normalized comma-separated `acceptTypes`, enabling multi-photo selection in Store Menu Gallery and Chat Gallery.
* **Team Chat Multi-Photo Upload:** Added `multiple` on `#team-chat-file-input` and updated `Vy()` to compress and send all selected images cleanly.

### 2. Store Menu Data-Loss Prevention & Live Catalog Recovery
* **Prevented Empty-Read Row Wipes:** Added `.select("data")` error checks across `set`, `update`, `remove`, multi-path `he()`, and chat metadata writers so transient read errors/timeouts abort instead of upserting `{}` and wiping store menu `imageUrl`s during `metaAttachmentId` write-backs.
* **Metadata Sync & Restored Menus:** Updated `fe()` to merge `smg__meta_index` and `directory.storeMenuMeta`, updated `gj()` to sync `storeMenuMeta` on page deletion, restored Jollibee's 6 menu pages (`smg__storesjollibee`), cleaned expired `404` links in `smg__wingscity` / `smg__yuvalfoodpark`, and resynced store menu counts/names.

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

