# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.1.7
- **Native Version**: 2.1.7 (versionCode 217)
- **APK Size**: 2381674 bytes (~2.27 MB)
- **SHA-256**: `63989d89391fe2ad8e65ac7b4df5970b783dffe59c20394ce3aedd348879094b`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Gallery Multi-Selection Upload**: Enables `Intent.EXTRA_ALLOW_MULTIPLE` in `MainActivity.onShowFileChooser()` when `MODE_OPEN_MULTIPLE` is used, normalizes comma-separated `acceptTypes`, and adds multi-image upload support to Team Chat.
  - **Store Menu Data-Loss Guard**: Checks `.select("data")` errors across `set`, `update`, `remove`, multi-path `he()`, and chat metadata writers so transient read failures cannot overwrite rows with `{}`.
  - **Menu Metadata & Catalog Recovery**: Merges `smg__meta_index` and `directory.storeMenuMeta` in `fe()`, syncs metadata on page deletes, and restores broken Jollibee (`smg__storesjollibee`), Wings City, Cafe Rafaelo, Potato Corner, and menu index entries.
