# Project notes

Living notes for the TV browser fork. Update this file when a decision or finding
affects later phases.

## Base project

- Upstream: `truefedex/tv-bro` (Kotlin, Android TV remote browser).
- Imported snapshot: `master` commit `ddffb8c231ff5dea382310148e3970e3a78bbd9c`
  (version `3.0.4`, versionCode `82`, dated 2026-11-07).
- Import method: source tarball
  `https://codeload.github.com/truefedex/tv-bro/tar.gz/refs/heads/master`
  (SHA-256 `7f2d2e39fe1be393e031e24271eca643654b8dbf5c98c2292db5925c1fac1f04`),
  then a fresh local git history. Upstream `.git` history is not imported.
- The GitHub fork/remote still has to be created (no credentials in this
  environment). The local layout is already the fork working tree.

## License (read before distributing)

`LICENSE.md` is NOT GPL. It is a BSD-style license with extra rules:

- Source redistribution must keep the copyright notice, conditions and disclaimer.
- Unmodified binary redistribution is allowed only if the binary is byte-identical
  to an official release published by the author on the upstream Releases page.
- Modified binary redistribution is allowed only if:
  - the app name does **not** contain "TV Bro" in any case;
  - the original app icon and the original `applicationId`
    (`com.phlox.tvwebbrowser`) are **not** used;
  - an About window in the GUI states that the app uses TV Bro's sources and
    links to <https://github.com/truefedex/tv-bro>.
- No warranty.

Implications for us:

- Keeping `LICENSE.md` and the source notices is required and already done.
- Private debug builds for testing are fine, but **any APK we share or release**
  must be renamed/re-branded and carry the About attribution. That is Phase 4
  work; do not publish an APK before it is done.
- Upstream's own `applicationIdSuffix` values (`.dev`, `.foss`, `.gecko`) are
  still based on the original id, so they do not satisfy the license for a
  modified public build. Plan: switch to our own id (for example
  `com.youcefsennoun.tvbrowser`) and our own name/icon in Phase 4.

## Modules, flavors, engines

Gradle modules:

- `:app` — application, TV and phone shells, settings, downloads, sync, player.
- `:app:common` — code and resources shared by engines and processes.
- `:app:gecko` — GeckoView engine implementation (only in `geckoIncluded`).
- `:adblock` — filter engine and lists.
- `buildSrc` — convention plugins (`tvbro.android.application` / `.library`).

Flavor dimensions:

- `appstore`: `generic` (default, built-in updater), `google`, `foss`.
- `webengine`: `geckoExcluded` (default, system WebView) and `geckoIncluded`
  (bundled GeckoView 157, separate app id suffix `.gecko`, `minSdk 26`).

Our target build: **`generic` + `geckoExcluded`**, i.e. the system WebView
engine. Reasons: Android 10 on the TX1-S has an updatable WebView; the Gecko
artifact is very large and heavy for 1 GB RAM / 8 GB storage. Keep the Gecko
flavor available but do not ship it by default.

## Build stack (actual upstream values)

- `compileSdk` = 37 (minor 1), `targetSdk` = 37, `minSdk` = 24.
  Android 10 is API 29, so the minimum requirement is satisfied.
- Gradle wrapper 9.9.0, Android Gradle Plugin 9.5.1, Kotlin 2.4.21,
  KSP 2.3.6, JDK toolchain 21.
- The roadmap mentions JDK 17; the current upstream stack pins JDK 21
  (`gradle/gradle-daemon-jvm.properties`, upstream release workflow). CI uses
  21 to match; changing it down is not a Phase 0 goal.
- Builds are **not** run on the phone/Termux (roadmap rule). CI produces the APK.

## Existing D-pad work already in upstream

The 3.x rewrite already covers much of roadmap Phases 3–4:

- TV shell with remote focus handling and a `VirtualCursorSupport` with cursor
  modes, plus video mode for fullscreen players.
- Central `BackNavigationEventsAdapter` with an overlay callback stack for
  predictable Back behavior.
- TV bar quick actions, address bar, voice search, bookmarks, downloads,
  history, settings, home page with tiles.
- Leanback launcher intent and TV banner/icon are present in the manifest.

This means Phase 3 is mostly **verification and tuning** on the real device, not
building these features from scratch. Home page tiles/settings (Phase 4) already
exist too; Phase 4 is mainly re-branding, defaults and translations.

## Reference projects

- `mrowser` (MIT): D-pad virtual cursor idea. The searched candidate is
  `m-salehi-v/mrowser` (MIT). Use as conceptual reference only. If we ever copy
  code, re-check that repository's license and keep its notice.
- `tuyafeng/Via`: verified via the GitHub API. Description says the repository
  is set up **for localization only**; it has no browser source. Do not use it
  as a base. Recorded as required by the roadmap.

## CI

- `.github/workflows/build.yml` (added in Phase 0) runs on push to `main`,
  pull requests and manual dispatch:
  - JDK 21 (Temurin), Ubuntu runner, existing Android SDK;
  - `./gradlew --no-daemon assembleGenericGeckoExcludedDebug`;
  - uploads the debug APK as the `tvbrowser-debug-webview` artifact.
- Debug variant has `applicationIdSuffix .dev`, so it can coexist with an
  installed official TV Bro.
- Upstream `release.yml` is kept untouched; it requires signing secrets we do
  not have yet (Phase 5).
- First green run confirmed: GitHub Actions run `43291101979` on `main`
  (2026-11-12), artifact `tvbrowser-debug-webview`, ~10.2 MB APK
  (`app-generic-geckoExcluded-debug.apk`). Local Gradle builds are intentionally
  not attempted.
- Repo: <https://github.com/YoucefSennoun/TV_Browser> (fork of `truefedex/tv-bro`,
  default branch `main`). Upstream `master` history is preserved in the fork.

## Small assumptions (roadmap rule 7)

- "Start building" = Phase 0 setup in one session: import base, notes, CI.
- We use the latest upstream `master` rather than an old tag; it has the
  modern module split and D-pad features described above.
- The workflow builds only the WebView flavor to keep CI time and APK size down;
  the roadmap's `assembleDebug` would build all six variants.
