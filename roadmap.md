# ROADMAP.md — D-pad Web Browser for Android TV Box

## 1. Goal
Build an Android app: a simple, fast web browser fully usable with a TV remote (D-pad: up/down/left/right/OK/Back). Easy for non-technical family members to use on a TV.

## 2. Target device
- Mortal TX1-S TV box, **1 GB RAM / 8 GB storage**, **Android 10 (API 29)**
- Input: remote with D-pad only (no touch, no mouse by default)
- Low resources: keep the app light (few tabs, no heavy libraries, small APK)

## 3. Base project
- **Fork `truefedex/tv-bro`** (Kotlin, built for TV remotes). Work in my fork, not the original.
- Before changing anything: read `LICENSE`, keep its notices, and note the license in `docs/NOTES.md`.
- Reference only (don't copy blindly; check its license first): `mrowser` (MIT) for its D-pad virtual cursor idea.
- Note: the `tuyafeng/Via` repo looks like a translations-only repo, not the full browser source. Do NOT use it as a base. Verify quickly and record the finding in `docs/NOTES.md`.

## 4. Environment and rules (IMPORTANT)
- Development happens in **Termux → proot Ubuntu** on a phone. No laptop.
- **Do NOT try to build the APK locally** (Gradle and the Android SDK are unreliable on ARM Termux).
- **Build with GitHub Actions**: every push to `main` builds a debug APK and uploads it as an artifact. I download it and sideload it on the TV box.
- Rules for the agent:
  1. Read the repo structure and this file first, and summarize your plan before editing.
  2. One small task per session. Do not touch unrelated files.
  3. Commit after each working step with a clear message. Never leave `main` with a failing build.
  4. No secrets, API keys, or keystores in the repo.
  5. Keep changes minimal and readable. Prefer existing project patterns.
  6. After each task, update the checklist in this file.

## 5. Phases

### Phase 0 — Setup
- [x] Fork and clone `tv-bro`; read the license and structure; write `docs/NOTES.md` (modules, build flavors, WebView vs GeckoView, minSdk/targetSdk). _Imported upstream master `ddffb8c` locally; GitHub fork/remote still needs credentials (see NOTES)._
- [x] Add `.github/workflows/build.yml`: JDK 17, Android SDK, `./gradlew assembleDebug`, upload APK artifact. _Uses JDK 21 and `assembleGenericGeckoExcludedDebug` because that is what the upstream build stack pins (see NOTES)._
- [ ] Fix any build errors until the workflow is green.
- **Done when:** a downloadable APK is produced from `main`.

### Phase 1 — Baseline on the device
- [ ] Make sure `minSdk` ≤ 29 and the app installs and launches on Android 10.
- [ ] Keep the leanback/TV launcher intent, banner and icon working.
- [ ] I install the unmodified build and report what is slow or broken. Record it in `docs/ISSUES.md`.
- **Done when:** the app opens, loads a page, and I can navigate with the remote.

### Phase 2 — Low-RAM tuning (1 GB)
- [ ] Limit simultaneous tabs (suggest max 3–4) and free memory on tab close.
- [ ] Disable or remove heavy or unused features and dependencies.
- [ ] Check WebView behaviour on Android 10; if the system WebView is too old for common sites, document the options (updated WebView or the project's alternative engine, if any).
- **Done when:** no crashes or out-of-memory kills during 15 minutes of normal browsing.

### Phase 3 — D-pad usability (core goal)
- [ ] Clear, high-contrast focus highlight on every focusable element.
- [ ] **Virtual cursor mode**: toggle with a key; D-pad moves a cursor with acceleration; OK clicks; edge-scrolls the page.
- [ ] Back key: close dialogs/menu → go back in history → exit confirmation. Predictable behaviour.
- [ ] Quick actions reachable by remote: Home, Back, Refresh, Zoom +/−, Desktop/Mobile user agent.
- [ ] Large URL bar and TV-friendly on-screen text input; keep voice search if it works.
- **Done when:** I can open a site, search, scroll, click small links and go back using only the remote.

### Phase 4 — Home screen and polish
- [ ] Home page with big tiles: bookmarks and most visited.
- [ ] Simple, large-text settings (default search engine, home page, user agent, clear data).
- [ ] App name, icon, TV banner; English UI first, add French/Arabic strings later.
- **Done when:** a family member can use it without help.

### Phase 5 — Release
- [ ] Release build workflow with a signed APK (keystore stored in GitHub Secrets, never in the repo).
- [ ] Write `README.md` with install steps and a GitHub Release `v0.1.0`.

## 6. Definition of done (v0.1.0)
- Installs on the TX1-S (Android 10) and launches from the TV launcher.
- Full browsing using only the remote.
- Stable on 1 GB RAM.
- CI builds an APK on every push.

## 7. Notes for the agent
- If a task is unclear, make the smallest safe assumption, write it in `docs/NOTES.md`, and continue.
- If something fails twice, stop, explain the cause in plain words, and propose two options instead of looping.
