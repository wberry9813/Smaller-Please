# Changelog

Notable changes to **Smaller, Please** (formerly ContextSlim). The product/Core version is the
release version; `contextslim` remains only a compatibility alias.

## [0.1.0-beta.7] - 2026-10-02

The first public release to bundle the Smaller, Please Agent Skill inside the Homebrew install.
Distributed through Homebrew; unsigned and DMG-free.

### Added

- **Bundled Agent Skill** — the canonical `skills/smaller-please/` bundle (`SKILL.md`,
  `workflows.md`, `reference.md`) now ships inside the release archive at
  `share/smaller-please/skills/smaller-please/` and is installed by the Homebrew formula, so users
  no longer need to clone the GitHub repository to get the Skill.
- **`smaller get skills`** — a new read-only command that resolves the bundled Skill relative to
  the installed executable (no hardcoded prefix) and prints the AI-agent installation instruction
  together with the `SKILL.md` content. It never writes to `~/.agents`, `~/.claude`, or any other
  AI-tool configuration directory, and never detects or installs an AI tool.

### Changed

- **Docs / website copy** — the public READMEs lead with `smaller get skills`, and the website
  install section now presents the simplified Skills-tab copy (website source only; see Known
  limitations).

### Known limitations / Planned

- The website Skills-copy update is **source-only / not deployed** at the time of this release.
- Claude Code live Skill acceptance remains **unverified** (the shared Agent Skills standard is
  documented as compatible).
- Paddle Live purchasing is not yet fully live pending merchant/domain verification; no real-card
  Production charge has been tested — only the 100% discount test path has been observed.
- Still **Planned**: automatic updates, the Chrome Web Store listing, and a public Media Pack
  download service. Intel (`x86_64`) and Windows are deferred; the Homebrew formula refuses Intel.

## [0.1.0-beta.6] - 2026-10-01

The first public beta whose release build includes the Production licence verification trust root.
Distributed through Homebrew; unsigned and DMG-free.

### Added

- **Production licence verification** — the normal release build now trusts the first Production
  licence verification key. A licence issued by the Production backend can activate Pro on the release build.

### Changed

- **Licence validation hardening** — the licence envelope's `key_id` is now bound to the signed
  payload; a mismatch is rejected.
- **Homebrew is the supported installation and update path** —
  `brew tap wberry9813/smaller-please` then `brew install smaller-please` /
  `brew upgrade smaller-please`. From this release, releases are unsigned and DMG-free; the
  earlier signed installer DMG is no longer produced.
- **Extension lifecycle honesty** — `smaller extension status` reports the staged extension's
  source/version and whether it is stale, and reports a "not staged" state honestly instead of
  implying the extension is loaded.
- **Homebrew onboarding** — the Homebrew formula's caveats and the install docs now guide the
  supported flow end to end: `brew install` → `smaller setup` → `smaller extension path` → Chrome
  "Load unpacked" from the visible `~/Applications/Smaller Please Extension` path (never from the
  Homebrew Cellar).

## [0.1.0-beta.5] - 2026-09-28

### Added

- **Canonical version contract** — all version surfaces are now strictly derived from `Cargo.toml`.
- **GitHub release automation** — draft-first release automation via `release.sh --publish github`.
- **Public Homebrew tap publication flow** — the tap formula is published from the released GitHub
  artifact.

### Changed

- **Pro capability delivery** — granular licence capabilities reach the CLI and the browser
  extension end to end.
- **Chrome extension Pro controls** — `Smart` stays selectable and the Pro-gated `Maximum` /
  metadata controls show a concise lock hint instead of silently changing state.

## [0.1.0-beta.4] - 2026-09-20

### Added

- **First public CLI macOS arm64 artifact** and public Homebrew formula (`brew tap wberry9813/smaller-please` + `brew install smaller-please`) shipped alongside the still-signed DMG.
- **AI-install fresh-user acceptance**.

### Changed

- **Version scheme** — added `version_scheme 1` in the Homebrew formula so betas upgrade over the historical `0.1.0`.

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
