# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains GitHub Actions CI/CD workflows for building and releasing Flutter mobile apps. There are no local build commands — all automation runs on GitHub Actions triggered by pushes to `main`.

## Workflow Triggering

Both workflows fire on push to `main` but are gated by commit message flags:

- Include `[android]` in the commit message to trigger the APK build
- Include `[ios]` in the commit message to trigger the IPA build
- Both flags can appear in the same commit message to trigger both

## Workflow Structure

`.github/workflows/apk-build-branch-and-release.yaml` — Android build, runs on `ubuntu-latest`  
`.github/workflows/ipa-build-branch-and-release.yaml` — iOS build, runs on `macos-latest`

Both workflows follow the same pattern:
1. Set `TAG_NAME` from `${GITHUB_REF_NAME}-${GITHUB_SHA::7}` (branch + short SHA)
2. Set artifact name: `App-${TAG_NAME}.apk` or `App-${TAG_NAME}.ipa`
3. Export both to `$GITHUB_ENV` for use in subsequent steps

Both require `permissions: contents: write` (needed for creating releases/tags).

## Current State

The workflows are stubs — they only set and echo variables. The remaining steps (checkout, Flutter SDK setup, build, artifact upload, GitHub Release creation) have not yet been added.
Typical next steps to complete each workflow:
- `actions/checkout` to clone the Flutter app repo
- `subosito/flutter-action` (or equivalent) to install Flutter
- `flutter build apk` / `flutter build ipa` with signing secrets
- `actions/upload-artifact` or `softprops/action-gh-release` to publish the artifact using `$APK_NAME` / `$IPA_NAME`
