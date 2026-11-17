# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.5.5
- **Native Version**: 2.5.5 (versionCode 255)
- **APK Size**: 47337216 bytes (~45.14 MB)
- **SHA-256**: `976785901375069c8e83927394d64d8f67f4d391b338ed2b7ba8996bf4953bc7`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Android "Install Unknown Apps" (`REQUEST_INSTALL_PACKAGES`) Gate & Auto-Resume**: Pre-install permission check redirects to Settings if unknown app installs are disabled; auto-resumes APK installation immediately upon returning from Settings without re-downloading.
  - **Native OpenStreetMap (osmdroid) Interactive Map Picker**: Embedded native OSM map picker with pin drop, GPS re-center, zoom controls, and multi-provider coordinate parser, eliminating Google Maps SDK dependencies and API keys.
  - **Inset-Safe Height-Bounded Modals & Pinned Action Bars**: Constrained dialog surfaces (`heightIn(max = 88% screen height)`) with pinned bottom action bars across Directory, Customer, and Address modals, eliminating button clipping on gesture navigation bars.
  - **Authoritative Commission Sync & Server-Verified Deletion**: Server scope replacement, tombstone filtering for voided records, and multi-tier Supabase cascade deletion.
  - **IME Soft Keyboard Auto-Scroll & Non-Blocking Media Dispatch**: Auto-scrolls composer and message list above soft keyboard; dispatches image captures and receipts asynchronously in background.
  - **Bi-Directional Message Reactions & Direct Facebook Page Sync**: Robust nested reaction parsing, reaction cache invalidation, and Cloudflare Worker sync relay.
