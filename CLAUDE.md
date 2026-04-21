# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository is a Flutter app with a GitHub Actions CI/CD pipeline that builds for all platforms and publishes GitHub Releases on every push to `main`.

## Local development

```
flutter run        # run the app
flutter test       # run tests
flutter build apk  # build Android (requires Android SDK)
```

The Flutter app lives at the repository root (`pubspec.yaml`, `lib/`, `android/`, `ios/`, etc.).

## Workflow pipeline

Single workflow: `.github/workflows/build-and-release.yaml`

**Job execution order:**

```
setup
  ├── android  (ubuntu-latest)
  ├── ios      (macos-latest)
  ├── web      (ubuntu-latest)
  ├── linux    (ubuntu-latest)
  ├── macos    (macos-latest)
  └── windows  (windows-latest)
        └── finalize
              ├── publish-android
              ├── publish-ios
              ├── publish-web
              ├── publish-linux
              ├── publish-macos
              └── publish-windows
```

**`setup` job** — computes `tag_name` (`$GITHUB_REF_NAME-$GITHUB_SHA::7`) and `release_name` (`App-$TAG_NAME`), exposed as job outputs consumed by all downstream jobs.

**Build jobs** — each checks out the repo, installs Flutter stable via `subosito/flutter-action@v2`, builds for its platform, and uploads a zipped artifact via `actions/upload-artifact@v4`:

| Job | Runner | Build command | Artifact path |
|-----|--------|--------------|---------------|
| android | ubuntu-latest | `flutter build apk --debug` | `build/app/outputs/flutter-apk/app-debug.apk` |
| ios | macos-latest | `flutter build ipa --no-codesign` | `build/ios/archive/` (xcarchive, no IPA — codesign not configured) |
| web | ubuntu-latest | `flutter build web` | `build/web/` |
| linux | ubuntu-latest | `flutter build linux` (requires `ninja-build libgtk-3-dev`) | `build/linux/x64/release/bundle/` |
| macos | macos-latest | `flutter build macos` | `build/macos/Build/Products/Release/` |
| windows | windows-latest | `flutter build windows` | `build/windows/x64/runner/Release/` |

**`finalize` job** — runs after all 6 builds complete; echoes `tag_name` and `release_name`.

**Publish jobs** — run in parallel after `finalize`; each downloads its artifact and creates/updates the GitHub Release named `App-$TAG_NAME` using `softprops/action-gh-release@v2`. Require `permissions: contents: write`.
