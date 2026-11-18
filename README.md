# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.6.0
- **Native Version**: 2.6.0 (versionCode 260)
- **APK Size**: 47371688 bytes (~45.18 MB)
- **SHA-256**: `841733e49c7b00881c6ec4a4b8abd56269a5cf2078ca238cc0a761a73f6f5a14`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Rider-Scoped Cart Icon Visibility & Ownership Guard**: Renders the green Cart / Receipt Wizard icon strictly on customers catered by the current rider using ID-first ownership checks (`ChatMeta.isCateredByRider`) with exact normalized name fallback, completely hides the Cart icon on other riders' cards (displaying `"Catered by <Rider>"`), and guards Cart click handlers (`openCartForChatGuarded`) against opening or overwriting another rider's active customer cart.
  - **Storage NoSuchBucket Fix & Worker Base64 Relay Fallback**: Unified `chat-media` bucket resolution with fallback probing (`chat-media` → `receipts` → `uploads` → `media`), direct compressed base64 JPEG Cloudflare Worker fallback for photos/receipts, and unified tap-to-retry across all attachment types.
  - **Active Chat Done-Jump Fix**: Cleared all completion fields across `customer_chat_meta` and `customer_chats` metadata on active/reopen transitions and enforced explicit non-done folder precedence.
