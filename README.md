


Capy Reader - spec.md  
last modified (utc): 2026-11-21-15-36-13  
instructions: on modify, update string.  
Note: created 2026-11-21 to record the Android 8-9 compatibility changes. Not a complete project spec.  


- fork from https://github.com/jocmp/capyreader, with android 8 and higher support.
- newer sqlite lib added, since older device lack appropriate version
- apl location (since it is bigger than 25 MB): https://drive.google.com/drive/folders/1vEOqcKNuHhUM2es591tybLCRHE-8g6GT?usp=sharing
- to download sources, click on 7z file above, then on arrow pointing down (on the right part of the screen) to download it.
- GPL-3.0 license, since that is the original app license



# 1. Minimum Android version and compatibility

## 1.1 minSdk
- minSdk is 26 (Android 8.0), set in `app/build.gradle.kts`. It was 30 before.
- targetSdk 36, compileSdk 37 (unchanged).
- Lint (`NewApi`, `InlinedApi`, `ObsoleteSdkInt`) is the check used for framework API calls at minSdk 26.

## 1.2 Image decoding (API 26-27)
- `MainApplication.newImageLoader` uses `AnimatedImageDecoder.Factory()` on API 28+ and `GifDecoder.Factory()` (from `coil-gif`) below that.

## 1.3 Display cutout (API 27+)
- `windowLayoutInDisplayCutoutMode` is set in `res/values-v27/themes.xml` only.
- `res/values/themes.xml` keeps the same styles without that attribute, so API 26 has no lint error.

## 1.4 Database engine (SQLite version)
- The schema needs SQLite 3.25+: migrations 20 and 22 use `ALTER TABLE ... RENAME COLUMN`, and several queries use UPSERT (3.24+).
- Android's built-in SQLite depends on the Android version and the manufacturer. Stock Android 9 has 3.22, so the app failed on first launch on a Kindle (Android 9) with `near "COLUMN": syntax error`.
- `AndroidDatabaseProvider.build()` checks `select sqlite_version()` on an in-memory database. If the version is below 3.25 it uses `io.requery.android.database.sqlite.RequerySQLiteOpenHelperFactory` from `com.github.requery:sqlite-android:3.45.0` (JitPack, declared in `gradle/libs.versions.toml`).
- The check runs once per app process (`by lazy`), not once per installation. If it fails, the built-in SQLite is used.
- Devices with 3.25+ keep the built-in SQLite. The bundled library adds roughly 600 KB per CPU architecture to the APK.

# 2. Test status
- Confirmed by the user: build succeeds (`assembleFreeDebug`); adding a local account works on Android 10 (phone) and on a 10-inch Kindle (Android 9) after the SQLite change.
- Not tested: Android 8.x, the GIF fallback, the API 27 cutout theme, release (R8) builds, migrations on an existing database created by an older app version.
- Lint still fails at `:app:lintFreeDebug` because of pre-existing errors (about 365 `MissingTranslation`, 113 `Untranslatable`, 3 `RestrictedApi` in `MainActivity.kt`). Those are not related to minSdk.

# 3. Known open items
- `OPMLImportWorker.kt` uses `FOREGROUND_SERVICE_TYPE_DATA_SYNC` (API 29 constant) at minSdk 26. Lint warns (`InlinedApi`). Not changed.
- README still says "Android 11 or later".
- `res/mipmap-anydpi-v26` is unnecessary at minSdk 26 (`ObsoleteSdkInt`). Not changed.
- `windowSizeBytes` and WAL behaviour with the bundled SQLite library are not verified on device.
- Lint baseline or `abortOnError` decision is still open.

# 4. Files changed for Android 8-9 support
- `app/build.gradle.kts`: minSdk 26; dependency `libs.requery.sqlite.android`.
- `gradle/libs.versions.toml`: `requery-sqlite-android` entry.
- `app/src/main/java/com/capyreader/app/MainApplication.kt`: GIF decoder fallback.
- `app/src/main/java/com/capyreader/app/common/AndroidDatabaseProvider.kt`: runtime SQLite check and bundled factory.
- `app/src/main/res/values/themes.xml`: cutout attribute removed.
- `app/src/main/res/values-v27/themes.xml`: new file, cutout attribute for API 27+.
