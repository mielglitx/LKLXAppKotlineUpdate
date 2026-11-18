# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.5.9
- **Native Version**: 2.5.9 (versionCode 259)
- **APK Size**: 47371680 bytes (~45.18 MB)
- **SHA-256**: `5da1d7c07f781fefde76bf52755d763c9577fe27937a7331ed412b25f508716e`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Fixed `Storage HTTP 400: Bucket not found (NoSuchBucket)` Across All Attachments**: Standardized primary Supabase Storage bucket `chat-media` with automatic fallback probing across `chat-media` → `receipts` → `uploads` → `media` and explicit per-type `Content-Type` headers across gallery images, camera photos, receipts, videos, voice notes, and documents.
  - **Direct Worker Base64 Relay Fallback for Photos & Receipts**: When Supabase Storage buckets are unavailable, compresses photos/receipts to JPEG (max 1080px, quality 80) and sends `data:image/jpeg;base64,...` directly to Cloudflare Worker `/api/send-reply` for multipart binary delivery to Meta Graph API.
  - **Unified Tap-to-Retry Across All Media Types**: Retains cached local `file://` files in `cacheDir/outbound_media/` on failure and cleanly transitions `"failed"` → `"sending"` → `"sent"` on retry across photos, receipts, videos, voice notes, and documents.
