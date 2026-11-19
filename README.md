# IRAQI TV – Native Android TV Streaming Application

**IRAQI TV** is a native Android TV and Google TV streaming application built with **Kotlin**, **Jetpack Compose for TV**, and **AndroidX Media3 ExoPlayer**, designed specifically for a 10-foot television experience with full **Arabic (RTL)** and **English (LTR)** support.

---

## Architecture & Project Structure

```text
app/src/main/java/com/example/
├── data/
│   ├── local/
│   │   ├── AssetChannelLoader.kt         # Local fallback channels.json parser
│   │   └── UserPreferencesDataStore.kt   # DataStore for favorites, recents, language & playback settings
│   └── remote/
│       ├── ChannelConfigApi.kt           # Retrofit endpoint for GitHub-hosted JSON config
│       ├── ChannelDto.kt                 # Moshi DTOs & domain mappers
│       └── NetworkModule.kt              # OkHttp + Retrofit + Moshi configuration
├── models/
│   ├── Channel.kt
│   ├── ChannelCategory.kt                # News, Sports, Religious, General Entertainment, Cultural, Regional
│   ├── PlaybackPreferences.kt
│   └── StreamProtocol.kt                 # HLS, DASH, YOUTUBE_OFFICIAL, MP4, NONE
├── navigation/
│   ├── AppRoutes.kt                      # @Serializable type-safe Navigation Compose routes
│   └── TvNavHost.kt                      # TV rail + screen navigation host
├── player/
│   ├── PlayerManager.kt                  # Media3 ExoPlayer HLS/DASH controller & error mapper
│   ├── PlayerUiState.kt
│   └── TvVideoPlayer.kt                  # AndroidView PlayerView surface
├── repositories/
│   ├── ChannelRepository.kt              # Remote JSON -> cached JSON -> bundled fallback
│   └── UserPreferencesRepository.kt
├── settings/
│   ├── SettingsScreen.kt                 # Language, sync, recents, buffer mode, version, about
│   └── SettingsViewModel.kt
├── ui/
│   ├── components/                       # ChannelCard, FeaturedHeroBanner, CategoryChipRow, TvNavigationRail
│   ├── screens/                          # HomeScreen, FavoritesScreen, ChannelDetailScreen, SearchScreen, PlayerScreen
│   ├── theme/                            # Compose for TV Material 3 dark theme + Cairo & IBM Plex Sans Arabic
│   └── viewmodel/                        # HomeViewModel, SearchViewModel, PlayerViewModel
└── utils/
    ├── AppStrings.kt                     # Bilingual Arabic (RTL) / English (LTR) strings
    ├── ExternalIntents.kt                # Official YouTube & broadcaster website launcher
    └── NetworkMonitor.kt                 # ConnectivityManager StateFlow
```

---

## Dynamic Channel Management (GitHub Remote JSON)

Channel listings are decoupled from the UI and managed via `ChannelRepository`:

1. **Remote Config**: Fetches JSON from `UserPreferencesState.DEFAULT_REMOTE_CONFIG_URL` (`https://raw.githubusercontent.com/.../channels.json`).
2. **Offline Cache**: Valid remote responses are cached in DataStore for fast startup and offline resilience.
3. **Bundled Fallback**: `app/src/main/assets/channels.json` provides the verified 13-channel Iraqi directory if the remote endpoint is unavailable.

See `sample_channels.json` in the repository root for the exact JSON schema. Administrators can:
- Add or remove channels
- Update `logoUrl` and `brandColorHex`
- Populate verified official `.m3u8` (HLS) or `.mpd` (DASH) `streamUrl` values
- Update `category` (`NEWS`, `SPORTS`, `RELIGIOUS`, `GENERAL_ENTERTAINMENT`, `CULTURAL`, `REGIONAL`)
- Toggle `isActive` to enable or disable a channel without releasing a new APK

> **Stream Integrity Note**: Channels without a verified live `streamUrl` clearly display **`البث غير متوفر حالياً`** and provide D-pad accessible links to the broadcaster's official YouTube channel and official website.

---

## Android TV Remote Controls

- **D-Pad Up / Down / Left / Right**: Navigate rails, hero actions, categories, channel rows, search filters, and settings.
- **D-Pad Center / Enter**: Open channel details, start playback, or summon the player OSD overlay.
- **Channel Up / Channel Down**: Switch to the next or previous Iraqi channel directly inside the player.
- **Play / Pause**: Toggle stream playback.
- **Back**: Dismiss player OSD overlay first, or return to the channel guide.

---

## Setup & APK Build Instructions

### Prerequisites
- JDK 17+
- Android SDK (API 36)
- Gradle

### 1. Build a Debug APK
```bash
gradle :app:assembleDebug
```
Output APK:
`app/build/outputs/apk/debug/app-debug.apk`

### 2. Run Unit & Robolectric Tests
```bash
gradle :app:testDebugUnitTest
```

### 3. Build a Signed Release APK
Set your keystore environment variables before running the release build:
```bash
export KEYSTORE_PATH=/path/to/your-release-key.jks
export STORE_PASSWORD=your_store_password
export KEY_PASSWORD=your_key_password

gradle :app:assembleRelease
```
Output APK:
`app/build/outputs/apk/release/app-release.apk`

### 4. Install on Android TV / TV Box via ADB
```bash
adb connect <TV_IP_ADDRESS>:5555
adb install -r app/build/outputs/apk/debug/app-debug.apk
```
