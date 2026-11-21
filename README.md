# Lokalex Kotlin Native App Update Repository
Repository for Base APK installer releases and release changelog for Lokalex Kotlin Native.

- **Version**: 2.8.9
- **Native Version**: 2.8.9 (versionCode 289)
- **APK Size**: 47731783 bytes (~45.52 MB)
- **SHA-256**: `9661b823c44493a481324dd3b67536d1afd3853b67170078dedf01a5a8f7a1a2`
- **Update Mode**: Pure Base APK Installer Only (`isApkOnly: true`, `forceApk: true`)
- **Latest Release Highlights**:
  - **Offline Philippine PSGC Location Gazetteer & Dynamic Hub Resolution**: Bundled a 132 KB compact offline PSGC hierarchy asset (`PhLocationGazetteer.kt`, `ph_psgc_locations.json`) covering all 18 official Philippine regions, 486+ cities/municipalities, and complete official barangay lists across operational provinces (0 network bytes). Hardened `getMainHub()` resolution hierarchy with Camiling fallback (never returning `"Hub"`), and pinned Region III and Region I at the top of the region selector.
  - **Barangay Name Guard in Municipality Dropdowns**: Added `PhLocationGazetteer.isBarangayName()` guard in `DirectoryAndRatesRepository.getDynamicMunicipalities()` preventing barangay names (such as "Anoling 1st" and "Anoling 2nd") from appearing in the Destination Municipality / City dropdown and quick-chips.
  - **Strict Cascading Isolation & Immediate Stale Barangay Reset**: Strictly filtered both PSGC gazetteer and custom records by `selectedRegion` (never leaking `serviceMunicipalities` or other-region records into MIMAROPA or international selections). Unconditionally resets `destinationBarangay = ""` whenever `destinationMunicipality` changes in `DirectoryRecordFormView` via dropdown or chip clicks.
  - **Security PIN RPC Propagation & Single-Tap Reactive Logout**: In `SecurityDialog`, awaited `saveRiderProfileFields()` with canonical ID resolution (`set_account_password_rpc`) before dismissing or showing toast. Replaced stale `savedName` with reactive state in `MainScreen.kt` so a single logout tap immediately clears state and returns to the login screen.
