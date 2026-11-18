# Lokalex App Release Notes

## Version 2.5.9 (2026-11-18)

### 1. Fixed `Storage HTTP 400: Bucket not found (NoSuchBucket)` Across All Attachments
* **Unified Supabase Storage Bucket Resolver & Fallback Probing (`MediaUploadRepository.kt`):**
  * Standardized primary Storage bucket `chat-media` with automatic fallback probing across `chat-media` → `receipts` → `uploads` → `media` whenever Supabase returns HTTP 404, `"Bucket not found"`, or `NoSuchBucket`, caching the working bucket for subsequent uploads.
  * Routed **all** attachment types—gallery images, camera photos, generated receipts, store menus, videos (`video/mp4`), voice/audio notes (`audio/mp4`), and documents (`application/pdf`, etc.)—through this exact resolver with proper per-type `Content-Type` headers and `POST`/`PUT` fallback.
* **Direct Cloudflare Worker Base64 Relay Fallback for Images & Receipts (`MediaUploadRepository.kt`):**
  * If all Supabase Storage buckets fail (e.g., `NoSuchBucket`, policy, or network block) when sending a photo, camera capture, or receipt, automatically compresses the image to JPEG (max 1080px, quality 80) and relays `data:image/jpeg;base64,...` directly to `POST /api/send-reply` (using the Worker's multipart binary upload to Meta Graph API) so customers receive photos and receipts immediately without getting blocked.

### 2. Unified Tap-to-Retry & Bubble Status Transitions
* **Cached Local Outbound Media Retry (`MediaUploadRepository.kt`, `LokalexRepository.kt`, `ChatConversationPane.kt`, `ChatFeedScreen.kt`):**
  * Preserves local `file://` files inside `context.cacheDir/outbound_media/` on failed photo, receipt, video, voice, and document bubbles so tapping **Tap to retry** re-runs upload and Worker dispatch cleanly.
  * Added `retryFailedAttachmentMessage` and hardened `reconcileAndCoalesceMessages` + `FailedMessageRetryBadge` visibility so every retried bubble transitions reliably from `"failed"` → `"sending"` → `"sent"`.

## Version 2.5.8 (2026-11-18)

### 1. Fixed Active & Unfinished Chat Threads Jumping Back to the Done Folder
* **Full Completion-Field Cleanup Across Supabase & Local State (`ChatMeta.kt`, `ChatRepository.kt`, `ChatFeedScreen.kt`, `RosterAndShiftRepository.kt`):**
  * Whenever a thread is reopened by a customer message, moved via **Undone**, assigned to **Catering**, or moved to **Inbox / Scheduled (Follow-Up) / Applicant / Stores**, all completion fields (`completedAt`, `completedBy`, `completedByRiderId`, `completedByRiderName`, `lastMarkedDoneAt`, `is_done`, `isDone`) are explicitly cleared (`null` / `false`) with `status = "active"` across both `customer_chat_meta` **and** `customer_chats` (`${custId}_metadata` rows), as well as local `ChatMeta` state and `lokalex_done_guard_prefs`.
* **Explicit Non-Done Folder Precedence During Sync & Polling (`ChatRepository.kt`, `RosterAndShiftRepository.kt`, `ChatFeedScreen.kt`, `ChatThreadListPane.kt`):**
  * Threads with an explicit active folder (`inbox`, `catering`, `followup`/`scheduled`, `applicant`, `stores`) and `status != "completed"` are never auto-coerced into `"done"` by leftover `completedAt`, legacy `category = "Done"`, or expired local done-guard timestamps (local done-guard is now strictly bounded to a 15-second post-Done grace window).
  * Fixed `ChatFeedScreen.kt` state sync (`chatMetaMap` and `selectedChat`) so reopened non-done threads from Supabase immediately replace stale local `Done` state instead of staying trapped in Done.
  * Prevented un-owned `"catering"` threads from being coerced into `"done"` in `reconcileChatsWithRoster` (returning them cleanly to `"inbox"` / `"active"`).
  * Fixed duplicate PSID alias merging (`<psid>` vs `fb_<psid>`) so a newer active row (e.g., after Undone or a rider/customer reply) beats an older stale `done` row.
* **Reliable Customer Auto-Reopen & Active Outbound Preservation (`ChatRepository.kt`, `ChatFeedScreen.kt`):**
  * New customer replies (`timestamp > completedAt`, or arriving while `folder == "done"`) automatically reopen threads to `folder = "inbox"`, `status = "active"`, `completedAt = null` across local state and Supabase.
  * Outbound rider text/media replies in `"inbox"` or `"catering"` preserve the active folder and `status = "active"`, never flipping an active thread to `"done"`.

## Version 2.5.7 (2026-11-18)

### 1. Outbound Message, Photo, Camera & Receipt Sending Fixes
* **Worker `/api/send-reply` & Canonical Room Normalization (`ChatRepository.kt`, `ChatFeedScreen.kt`):**
  * Normalized outbound PSIDs cleanly: strips `fb_` / `messenger_` prefixes for Cloudflare Worker `recipientId` while always persisting Supabase `customer_chats` rows under canonical `room_id = "fb_<psid>"`.
  * Attaches active `apiId` from `facebookApis` / `facebook_channels` alongside explicit `Content-Type: application/json` headers, omits blank `text` fields on media payloads, and sanitizes `replyToMid` to only send valid Meta `mid.$...` / `m_...` IDs (never sending `"null"` or local/temp IDs).
  * Moved `dispatchTextMessage` onto `mediaDispatchScope` (`SupervisorJob`) so closing dialogs or switching tabs never cancels in-flight sends, and replaced the overly broad send lock with identical-message 1200ms debounce so Insta-Reply & back-to-back messages always send.
* **Synchronous `cacheDir/outbound_media/` Media Caching (`MediaUploadRepository.kt`, `ChatFeedScreen.kt`, `CartReceiptScreen.kt`, `ChatRoomComponents.kt`):**
  * Synchronously copies selected gallery `content://` URIs, camera/editor bitmaps, and rendered receipt JPEGs into `context.cacheDir/outbound_media/` **before** closing modals or launching background uploads—eliminating expired URI `SecurityException` and `FileNotFoundException`.
  * Upgraded Supabase Storage uploads with explicit `Content-Type`, automatic `POST` + `PUT` retry, and strict public HTTPS URL enforcement for Messenger threads.
  * Added `CoroutineExceptionHandler` and full `try / catch` error surfacing across every background upload queue: failures now log exact details via `AppErrorTracker`, mark the bubble `"failed"` with 1-tap retry in `ChatMessageLocalCache` and UI, and show a visible error Toast so failures are never silent.

### 2. Fixed Duplicate Image Thumbnails in Single-Image Bubbles
* **Base-Path & Facebook CDN URL Deduplication (`ChatMeta.kt`, `ChatRepository.kt`, `ChatConversationPane.kt`, `ChatRoomComponents.kt`):** Deduplicates `images` by exact URL, normalized base path (`substringBefore("?").substringBefore("#")`), and Facebook CDN `asset_id` / filename keys so `image_data.url` vs `payload.url` or CDN edge-host copies never render duplicate thumbnails.
* **Clean Optimistic-to-Server Image Replacement (`ChatMessageLocalCache.kt`, `ChatRepository.kt`, `ChatFeedScreen.kt`, `MediaUploadRepository.kt`):** Replaced naive list concatenation with `ChatMessage.resolveMergedImageUrls`, cleanly replacing optimistic `data:` / `file://` / `temp_...` Supabase previews with confirmed `mid_...` server images without producing 2–3 thumbnails for a single sent photo.

### 3. Preserved Rider Sender Name on Outbound Messages (Stopped Reverting to `[Page]` / `Page Admin`)
* **Immediate `mid` & Rider Attribution Linking (`ChatRepository.kt`, `MediaUploadRepository.kt`):** Immediately patches outbound Supabase rows and local SQLite cache entries with `mid` / `metaMid`, `sender = riderName`, `senderType = "rider"`, `isFromPage = false`, and `isPageEcho = false`, backed by an in-memory ±20s attribution registry.
* **Protected Rider Attribution Across Sync & Merge (`ChatMeta.kt`, `ChatMessageLocalCache.kt`, `ChatRepository.kt`, `ChatFeedScreen.kt`, `ChatConversationPane.kt`):** Known rider names now always win over `"Page Admin"`, `"Facebook Page"`, `"Lokalex Delivery"`, and `is_echo` webhook rows regardless of arrival order, while genuine direct Facebook Page replies keep the `[Page]` badge.

### 4. Insta-Replies Cleanup
* **Removed Hardcoded Built-In Templates (`LokalexRepository.kt`, `ChatFeedScreen.kt`):** Removed hardcoded `Cater Greeting`, `Closing Message`, and default `On the way` / `Nandito na` / `Papunta na` templates so **Insta-Replies** displays only your saved custom Supabase replies—each with **Edit (✏️)** and **Delete (trash)** buttons.

## Version 2.5.5 (2026-11-17)

### 1. Android "Install Unknown Apps" (`REQUEST_INSTALL_PACKAGES`) Pre-Install Gate & Auto-Resume
* **Pre-Install Permission Guard (`ApkUpdateDownloader.kt`):** Before firing package installation intents on Android 8.0+ (API 26+), verifies `packageManager.canRequestPackageInstalls()`. If missing, preserves the downloaded `.apk` path in `pendingApkFile` and smoothly redirects to `Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES` (`package:${context.packageName}`).
* **Auto-Resume Without Re-Download (`MainActivity.kt`):** Registered an `ActivityResultLauncher` (`StartActivityForResult`) to intercept when the user returns from system Settings. Combined with `onResume()` verification, automatically resumes and launches the Android package installer prompt for the downloaded APK as soon as permission is granted.
* **Startup & In-Dialog Permission Checklist (`BackgroundPermissionHelper.kt`):** Added `canInstallUnknownApps` and `openInstallUnknownAppsSettings` helpers, surfacing a 1-tap **Enable** setup banner on `MainScreen.kt` and an in-dialog alert badge inside `AppUpdateDialog` so riders can proactively enable APK update permissions.

### 2. Native OpenStreetMap (osmdroid) Interactive Map Picker
* **Full Google Maps SDK Replacement:** Replaced Google Maps SDK and WebView Leaflet HTML implementations with native `org.osmdroid.views.MapView` embedded via Compose `AndroidView`, eliminating proprietary Play Services map dependencies and API keys.
* **Interactive Map Controls & Visual Pin (`OsmMapPickerDialog`):** Features interactive zoom controls (+ / −), live GPS re-center (🎯) with loading indicator, floating center pin (`📍`) with pinpoint target, and real-time pinned coordinate readouts.
* **Universal Multi-Provider Coordinate Parser (`LokalexLocationHelper.kt`):** Robustly parses coordinates from OSM links (`mlat`/`mlon`), Google Maps queries (`q=`, `@`, `loc:`), Apple Maps, `geo:` URIs, and raw `lat,lng` strings. Formats standard OpenStreetMap marker URLs using `Locale.US`.
* **Screen Integrations:** Embedded across Customer/Store Directory (`DirectoryDialog.kt`), Cart & Receipt Wizard address field (`CartReceiptScreen.kt`), and Chat composer `location pin` chip (`ChatConversationPane.kt` & `ChatFeedScreen.kt`).

### 3. Inset-Safe Height-Bounded Modals & Pinned Action Bars
* **Touch-Safe Button Visibility:** Constrained dialog surfaces across `DirectoryDialog.kt`, `CreateCustomerDirectoryDialog`, `AddAddressModal`, and `AddCustomerModal` using `Modifier.fillMaxWidth(0.95f).heightIn(max = screenHeightDp * 0.88f).navigationBarsPadding().imePadding()`.
* **Three-Part Column Architecture:** Separated layouts into a pinned top header, scrollable middle content (`LazyColumn` or `verticalScroll`), and a pinned bottom action bar (`heightIn(min = 48.dp)`), completely eliminating button clipping and unclickable elements on gesture navigation devices.

### 4. Authoritative Commission Sync & Server-Verified Deletion
* **Authoritative Server Scope Replacement:** Replaces cached records with the authoritative server list returned for the queried date, pruning deleted rows rather than naively appending duplicates.
* **Persistent Deleted-ID Tombstones:** Implemented `deletedReceiptTombstones` in `SharedPreferences` filtering voided and soft-deleted records from rider gross totals, net take-home, and customer breakdowns.
* **Multi-Tier Delete Mutations:** Executes hard Supabase `DELETE` with soft-delete fallback (`status: "VOIDED", is_deleted: true`), clearing linked entries in `catered`, `catered_history`, and `commission_penalties`.

### 5. IME Soft Keyboard Resize & Non-Blocking Media Dispatch
* **`adjustResize` & Auto-Scroll:** Applied `imePadding()` to the chat composer while avoiding double navigation bar padding when keyboard is up. Automatically scrolls to the latest message upon keyboard display or text field focus.
* **Lifecycle-Independent Media Queue:** Created a background `mediaDispatchScope` decoupled from Compose dialog lifecycles, immediately creating optimistic bubbles and dismissing modals while image upload and receipt generation process in the background.

### 6. Bi-Directional Message Reactions & Direct Facebook Page Sync
* **Nested & Flat Reaction Parser:** Correctly parses Cloudflare Worker reactions (`{"❤️": {"<senderId>": true}}`) alongside flat emoji maps, cleanly updating sender reaction counts.
* **Reaction Cache Invalidation:** Overwrites reactions, status, and edit flags on existing cached messages in `ChatMessageLocalCache.kt`.

## Version 2.4.9 (2026-11-17)

### 1. Instant Chat Card Category & Folder Reactivity
* **Immediate Frame-1 Card Relocation:** Resolved the issue where a chat card remained in its active category list after status changes (`Done`, `Cater`, `Undone`, `Followup` / `Scheduled`, `Applicant`, `Cancel`) until manually switching category tabs. Replaced the un-keyed `remember` in `ChatFeedScreen.kt` with a Compose `mutableStateMapOf<String, ChatMeta>()` (`chatMetaMap`) and keyed `derivedStateOf { chatMetaMap.values.map ... }`. Because reads on `chatMetaMap.values` automatically register snapshot observers, any folder or category transition removes the card from the active category list on frame 1 without needing tab switches.
* **Synchronous Main-Thread Optimistic State Machine:** Created `updateChatMetaOptimistic(targetId, transform)` in `ChatFeedScreen.kt`, executing immutable `.copy(...)` transformations directly on the main thread across all user actions (`executeMarkCustomerDone`, `onQuickCater`, `onQuickMarkUnread`, `onMenuCaterClick`, `onMenuUnreadClick`, `onMenuUndoneClick`, `onMenuApplicantClick`, `onMenuFollowUpClick`, and cancel order confirmation) before triggering asynchronous network operations.
* **Stable Thread Identity Pipeline (`psid`):** Added `val psid: String get() = customerId.ifBlank { id.removePrefix("fb_") }` in `ChatMeta.kt` and wired `items(displayChats, key = { it.psid.ifBlank { it.stableUniqueKey } })` in `ChatThreadListPane.kt` for guaranteed stable Compose item identity and smooth exit animations.
* **Reactive Category Header Counters:** Passed `chatMetaMap.values.toList()` to `ChatCategoriesHeaderBar` so category tab unread counters reflect optimistic moves immediately.

### 2. Facebook Page Message Synchronization & Non-Text Attachment Fix
* **Global Tail Overlap Sync (`fetchRecentGlobalMessages`):** Replaced strict `id=gt.$lastMaxId` increments with a 15-second overlap window (`timestamp=gt.(cursor - 15s)`) and added global tail polling of the latest 80 messages across all threads on 6-second ticks, merging idempotently into SQLite via `saveMessagesGrouped` across rooms.
* **Non-Text & Attachment-Only Message Preservation:** Prevented blank-text message drops in `parseChatMessageFromJson` by inspecting attachments for stickers (`🏷️ [Sticker]`), locations (`📍 [Location]`), links (`🔗 [Title]`), voice/video/file attachments, or generic fallback (`[Attachment]`).
* **Auto-Surfacing New & Returning Customers:** Synthesizes missing `ChatMeta` records into `Inbox` / `Unread` when new messages arrive from unindexed PSIDs; automatically reopens threads from `"Done"`, `"Cancelled"`, or `"Closed"` into `Inbox` / `Unread` whenever a customer message arrives after the completion boundary. Upgraded `customer_chat_meta` query limits to `limit=500`.
* **Force Message Update via Cloudflare Worker Relay:** Wired `POST https://dry-firefly-ee76.lokaledeliver.workers.dev/api/sync-messages` (`limitMessages = 20`, `limitConversations = 25` / `recipientId = cleanPsid`) coupled with full Supabase tail bypass (`limit = 100` for active thread, `limit = 200` global), infinite spinning icon feedback, and toast notification (`"✅ Synced latest messages from Facebook Page"`). Accessible via header 1-tap refresh button and the 3-dots chat menu (`• Force Message Update`).

## Version 2.4.8 (2026-11-17)

### 1. Ungated Done Flow
* **Unrestricted Order Completion:** Removed the receipt verification gate from customer `Done` buttons across both the **Rotations** tab (`RotationsScreen.kt`) and the **Chat menu** (`• Done` in `ChatFeedScreen.kt` & `ChatConversationPane.kt`). Riders and staff can now mark active customers `Done` immediately without being blocked by missing receipts.

### 2. Strict Receipt Lock on Void & Cancel with Admin/TL Override
* **Lock Against Accidental Order Annulment:** Strictly blocks regular riders from executing **Void** (`RotationsScreen.kt`) or **• Cancel** (`ChatFeedScreen.kt` / `ChatConversationPane.kt`) once a receipt has already been created for that customer/thread (`psid`, numeric ID, or customer name/alias). Regular riders receive a warning toast: `"🔒 Cannot void or cancel — a receipt has already been generated for [Customer Name]."`.
* **Authorized Admin & TL Annulment:** Administrators (`hasEffectiveAdminAccess`) and Team Leaders with force-status permissions (`canForceRiderStatus`) can override the lock. Upon confirmation, `repository.voidReceiptsForCustomer` automatically removes the linked receipt records from Supabase `receipts`, `catered`, and `catered_history`, purges active Cart Wizard drafts, and voids the catering assignment.

### 3. Instant Chat Done Optimistic Transition & 15-Second Grace Window
* **Frame-1 Folder Relocation:** Marking a thread `Done` immediately updates in-memory `ChatMeta` (`category = "Done"`, `folder = "done"`, `status = "completed"`, `cateredBy = null`, `isDone = true`) on frame 1, clearing it from `Inbox` and `Catering` and moving it to the `Done` folder without waiting for network round-trips.
* **15-Second Anti-Poll-Overwrite Protection:** Recorded thread IDs in `lastMarkedDoneAtByThread` guard against background delta polling and roster synchronization overwriting the thread back to `Catering` during the propagation grace window.
* **Auto-Message & Auto-Available Transition:** Sends the closing automated end-chat message, evicts the customer from `ActiveCateringLocalCache`, and checks remaining active bookings: if 0 active orders remain for the rider, automatically reverts their roster status to `Available` (`isForcedCater = true`).

### 4. Direct OkHttp Streaming APK Downloader
* **Fast Direct Connection:** Replaced slow system `DownloadManager` queuing with direct OkHttp streaming on `Dispatchers.IO` (`connectTimeout = 10_000`, `readTimeout = 30_000`, redirect follow enabled).
* **Live Progress & Throttled UI Updates:** Immediately transitions state to `isDownloading = true` and `progress = 0f` on tap, streaming 32KB chunks directly into `context.getExternalFilesDir(Environment.DIRECTORY_DOWNLOADS)/Lokalex-Update.apk` and emitting smooth byte progress (`150ms` throttle) to Compose UI.
* **Automatic Package Installation:** Automatically verifies APK archive integrity and invokes the Android package installer via `FileProvider` upon hitting 100%.

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

