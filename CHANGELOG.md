# Changelog

Notable changes to **Smaller Please** (formerly ContextSlim). The product/Core version is the
release version; `contextslim` remains only a compatibility alias.

## [0.1.0-beta.3] - Beta 3

The canonical product name is now **Smaller Please** (no comma), and the shipped installer app is
**`Smaller Please Installer.app`**.

### Distribution

- The macOS arm64 installer DMG
  (`Smaller-Please-Installer-0.1.0-beta.3-macos-arm64.dmg`) is **signed with a Developer ID
  Application certificate and notarized by Apple**, with the notarization ticket **stapled** to
  the DMG. Gatekeeper (`spctl`) reports the published artifact as `accepted`
  (`Notarized Developer ID`), and `xcrun stapler validate` succeeds (verified).
- `SHA256SUMS` and `release.json` are published next to the DMG so you can verify the download.
- `THIRD_PARTY_NOTICES.md` is published with the release for the optional LGPL Media Pack.

### Changed

- **Product naming** — the product is `Smaller Please` (no comma), and the installer app is
  `Smaller Please Installer.app`.

### Known limitations / Planned

- **Chrome Web Store listing is Planned.** Installing the extension requires the one-time manual
  Chrome **Developer mode → Load unpacked** step; this is Chrome's security model and cannot be
  automated.
- **A public Homebrew tap (`homebrew-smaller-please`) is Planned.** There is no public tap today.
- **Automatic updates are Planned.** There is no background auto-update; updating means running
  the newer installer and reloading the extension.
- **A public Media Pack download service is Planned.** The Media Pack remains a separate local
  artifact and is never downloaded automatically.
- **Intel (`x86_64`) and Windows are deferred.**
- Some website upload paths are not yet live-verified, and a few are verified `passthrough`
  (the original file is uploaded unchanged). These are labeled honestly in the extension; see
  `docs/EXTENSION_SETTINGS.md`.

## [0.1.0-beta.2] - Beta 2

The first publicly distributed Smaller Please beta, for **macOS Apple Silicon (arm64)**.

### Distribution

- The macOS arm64 installer DMG
  (`Smaller-Please-Installer-0.1.0-beta.2-macos-arm64.dmg`) is **signed with a Developer ID
  Application certificate and notarized by Apple**, with the notarization ticket **stapled** to
  the DMG. Gatekeeper (`spctl`) reports the published artifact as `accepted`
  (`Notarized Developer ID`), and `xcrun stapler validate` succeeds (verified).
- `SHA256SUMS` and `release.json` are published next to the DMG so you can verify the download.
- `THIRD_PARTY_NOTICES.md` is published with the release for the optional LGPL Media Pack.

### Added

- **Image optimization** — HEIC/JPEG/TIFF → JPEG and PNG/PNG-with-transparency handling, a
  never-grow guard, a minimum-savings threshold, metadata stripping, and a per-media minimum
  source size (a hard gate). Batch traversal excludes generated `.contextslim/` output.
- **Video optimization** — MP4/MOV/M4V → MP4 (H.264 + AAC, `+faststart`, metadata removal) with
  the same never-grow guard and thresholds.
- **Chrome extension (MV3)** — ChatGPT/Claude picker, drag-and-drop, and paste interception;
  local optimization through the Native Messaging host; a progress chip; and honest fallback to
  the original when a file is skipped or fails.
- **Native Core** — the `smaller` CLI (legacy `contextslim` alias), a single persisted config,
  a marker-validated managed store with cache/dedupe and storage statistics, and `clean`.
- **Media Engine** — the optional LGPL-only Media Pack (FFmpeg 7.1, macOS arm64, VideoToolbox
  H.264 + native AAC), with automatic fallback to an existing Homebrew/system FFmpeg.
- **macOS installer** — a `Smaller Please Installer.app` installer + DMG that installs Core, the
  extension bundle, and the Media Pack user-level (no `sudo`), and supports overwrite/upgrade.
- **Languages** — English + Simplified Chinese in the installer and extension (Follow system).
- **Diagnostics and repair** — `smaller doctor`, `smaller setup`, `smaller repair`, the
  extension/native/media lifecycle commands, and the `smaller uninstall` lifecycle
  (settings/cache kept by default).

### Known limitations / Planned

- **Chrome Web Store listing is Planned.** Installing the extension requires the one-time manual
  Chrome **Developer mode → Load unpacked** step; this is Chrome's security model and cannot be
  automated.
- **A public Homebrew tap (`homebrew-smaller-please`) is Planned.** There is no public tap today.
- **Automatic updates are Planned.** There is no background auto-update; updating means running
  the newer installer and reloading the extension.
- **A public Media Pack download service is Planned.** The Media Pack remains a separate local
  artifact and is never downloaded automatically.
- **Intel (`x86_64`) and Windows are deferred.**
- Some website upload paths are not yet live-verified, and a few are verified `passthrough`
  (the original file is uploaded unchanged). These are labeled honestly in the extension; see
  `docs/EXTENSION_SETTINGS.md`.

## [0.1.0] - Development

Local-first media optimization before images/videos are uploaded to AI agents or browsers.
Sources are never modified. macOS-first; this is the development feature line, and the
distribution builds above are signed and notarized.

### Added

- **Image optimization** — HEIC/JPEG/TIFF → JPEG and PNG/PNG-with-transparency handling, a
  never-grow guard, a minimum-savings threshold, metadata stripping, and a per-media minimum
  source size (a hard gate). Batch traversal excludes generated `.contextslim/` output.
- **Video optimization** — MP4/MOV/M4V → MP4 (H.264 + AAC, `+faststart`, metadata removal) with
  the same never-grow guard and thresholds.
- **Chrome extension (MV3)** — ChatGPT/Claude picker, drag-and-drop, and paste interception;
  local optimization through the Native Messaging host; a progress chip; and honest fallback to
  the original when a file is skipped or fails.
- **Native Core** — the `smaller` CLI (legacy `contextslim` alias), a single persisted config,
  a marker-validated managed store with cache/dedupe and storage statistics, and `clean`.
- **Media Engine** — the optional LGPL-only Media Pack (FFmpeg 7.1, macOS arm64,
  VideoToolbox H.264 + native AAC) installed with `smaller media install`, with automatic
  fallback to an existing Homebrew/system FFmpeg.
- **macOS installer** — a `Smaller Please Installer.app` installer + DMG that installs Core, the
  extension bundle, and the Media Pack user-level (no `sudo`).
- **Languages** — English + Simplified Chinese in the installer and extension (Follow system).
- **Diagnostics and repair** — `smaller doctor`, `smaller setup`, `smaller repair`, the
  extension/native/media lifecycle commands, and the `smaller uninstall` lifecycle
  (settings/cache kept by default).

### Known limitations / Planned

- Public distribution, automatic updates, the Chrome Web Store listing, a **public** Homebrew
  tap, and a **public** Media Pack download/signing service are **Planned**, not available in
  0.1.0. Intel (`x86_64`) and Windows are deferred.
