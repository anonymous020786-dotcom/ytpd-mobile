# Playlist Grabber — Mobile

Android and iOS wrapper for Playlist Grabber, built with [Capacitor](https://capacitorjs.com/). Unlike the [desktop build](https://github.com/anonymous020786-dotcom/ytpd-desktop), this is **not** a local sidecar app — neither Android nor iOS allows an app to spawn arbitrary native executables the way Electron does on desktop. Instead, this is a thin native shell: Capacitor's WebView loads the **hosted** [ytpd-web](https://github.com/anonymous020786-dotcom/ytpd-web) frontend directly, same as opening it in a mobile browser, just packaged as an installable app with a real icon, splash screen, and app-store presence.

**This means the app does nothing useful until `ytpd-web` is actually deployed somewhere reachable.** `capacitor.config.json`'s `server.url` is currently a placeholder (`https://your-domain-here.example.com`).

## Status

- Android and iOS projects are scaffolded, icon/splash assets generated, CI wired up.
- **Blocked on:** a live hosted deployment of `ytpd-web` (in progress separately — needs a fresh AWS account + Supabase Postgres project, both requiring the repo owner's own signup).
- **Blocked on, for real store distribution:** an Apple Developer Program membership ($99/yr) and a Google Play Console account ($25 one-time) — both need the repo owner's own identity/payment directly with Apple/Google. Nothing here can substitute for that.

## Setting the hosted URL

Once `ytpd-web` is deployed:

1. **Local builds:** edit `capacitor.config.json`'s `server.url` directly, then `npm run sync`.
2. **CI builds:** set a repository variable (Settings → Secrets and variables → Actions → Variables tab, not Secrets — it's just a URL, nothing sensitive) named `YTPD_HOSTED_URL`. The release workflow patches it into the config before building.

## Building locally

Requires Node 20+. Android additionally needs a JDK (17+) and the Android SDK (Android Studio's SDK Manager is the easiest way to get both); iOS needs Xcode and can only be built on macOS.

```bash
npm install
npx cap sync
```

**Android** (produces a debug APK, unsigned, fine for sideloading — not for Play Store):

```bash
npm run open:android   # opens Android Studio - Build > Build Bundle(s)/APK(s) > Build APK(s)
# or headless:
cd android && ./gradlew assembleDebug
```

**iOS** (requires a real Mac + Xcode; this repo/CI cannot build a real installable `.ipa` without an Apple Developer account — see Status above):

```bash
npm run open:ios       # opens Xcode
```

## CI: `.github/workflows/release.yml`

- **`android` job** — builds a real debug APK on `ubuntu-latest` and publishes it to this repo's Releases on a version tag push. Unsigned, so it's sideload-only, not Play Store ready.
- **`ios-build-check` job** — builds the Xcode project for the iOS Simulator on `macos-latest` with code signing disabled. This is a **correctness check only**, not a release: it proves the native project and web URL wiring actually work, but produces no distributable artifact, since a real `.ipa` needs signing credentials this project doesn't have.

## What's genuinely not done yet

- **Any real app-store presence.** Building the APK/IPA is necessary but nowhere near sufficient — Play Store and App Store listings, screenshots, privacy policy, content ratings, and (per earlier discussion in this project) real policy risk specific to YouTube-downloader-style apps, are all separate, unstarted work.
- **Native file-save handling.** The web frontend's download flow uses a browser `<a download>` trick that works in real browsers but is unreliable inside a Capacitor WebView. A proper fix means adding `@capacitor/filesystem` (+ probably `@capacitor/share`) and branching the frontend's download code on `Capacitor.isNativePlatform()` — not done yet, so downloads may not reliably save on-device until this is addressed.
- **Push notifications / background download progress** — not investigated at all.
- **iOS real-device builds** — genuinely impossible without the Apple Developer account (see Status).
