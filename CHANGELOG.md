# Changelog

Notable changes to **Smaller, Please** (formerly ContextSlim). The product/Core version is the
release version; `contextslim` remains only a compatibility alias.

## [0.1.0] - 2026-10-03

**The first stable release of Smaller, Please.** It consolidates the beta line into the supported
`0.1.0` product: a local-first media optimizer that shrinks images and video before they are
consumed by AI agents or uploaded in the browser, without modifying the original file.

### Shipped

- **Local image optimization** and **local video optimization** through the single public
  `smaller optimize <input>` command (image, video, or directory); `--mode smart` is the default
  safety pipeline and never keeps a derivative larger than its source.
- **Browser upload optimization** on the currently supported sites and paths: **ChatGPT**,
  **Gemini**, and **DeepSeek**. An upload is replaced with an optimized derivative only after a
  confirmed handoff, and fails open to the original otherwise. **Gemini drag/drop is
  passthrough** — the original attaches with no optimization claim.
- **Source preservation** — originals are never modified, overwritten, or deleted; optimization
  writes derivatives only, and a below-threshold source is skipped with the original used.
- **Managed Core store / cache** at `~/.contextslim-bridge/store`, with marker-validated
  `derivatives/`, `cache/`, and `tmp/`; `smaller clean` removes only marker-validated managed
  content and the bridge-owned `inbox/`.
- **`smaller prepare` / `prepare-batch`** — frozen JSON output with an authoritative `use_path` for
  agents; derivatives and cache default to the managed store rather than a source-adjacent
  `.contextslim/` directory.
- **Bundled Agent Skill** inside the Homebrew install
  (`share/smaller-please/skills/smaller-please/`) plus the read-only **`smaller get skills`**
  command, which surfaces the Skill and its install instruction without writing to any AI-tool
  directory. The corrected install/update lifecycle is documented: install the Skill via a stable
  symlink so `brew upgrade smaller-please` keeps it current; copying is a fallback that requires
  re-install after a future upgrade.
- **Free / Pro capability split** — Free covers the `smart` pipeline; Pro unlocks `maximum`
  compression and explicit metadata `remove-all`, resolved locally from a stored licence.
- **Production licence verification** — licences are verified offline on-device against the
  shipped Production trust root; a Pro licence is purchasable and activates locally, and unknown or
  unrecognized installs fail closed to Free. Media content is never uploaded to a Smaller, Please
  server.

### Installation / platform

- **Homebrew is the supported installation channel**:
  `brew tap wberry9813/smaller-please` then `brew install smaller-please`.
- **macOS 12+ on Apple Silicon (`arm64`) only**; the Homebrew formula refuses Intel.
- Installing the **browser extension is always manual**: Chrome **Developer mode → Load unpacked**
  from the visible `~/Applications/Smaller Please Extension` path staged by `smaller setup` /
  `smaller extension`.

### Not yet available

- Automatic updates, the **Chrome Web Store** listing, **Intel (`x86_64`)**, and **Windows** are
  not shipped. There is no signed DMG/installer: the earlier DMG is historical and no longer
  produced.

## [0.1.0-beta.8] - 2026-10-03

Corrects agent-facing `prepare` output placement and tightens the bundled Agent Skill, following
the first real-world Beta.7 Skill acceptance.

### Changed

- **Agent `prepare` / `prepare-batch` derivatives now default to the Core managed store** — optimized
  outputs are written to `<bridge_store_dir>/derivatives/` instead of a source-adjacent
  `.contextslim/` directory. `smaller optimize` output semantics are unchanged.
- **`prepare` cache/dedupe now lives under the managed store** — cache and dedupe records are stored
  in the managed store's `cache/` rather than beside the sources.
- **`--output` is preserved and overrides the derivative directory only** — an explicit
  `--output <dir>` keeps its existing behavior (it overrides where optimized derivatives are
  written); the cache remains in the managed store.
- **Bundled Agent Skill input-resolution contract** — the Skill now defines a safe order for
  resolving an attachment input (authoritative local filesystem path → standard attachment/file API
  → one-time byte materialization → ask the user) and forbids discovering attachments by crawling
  HOME, unrelated folders, application/browser/agent caches, private databases, or broad temp
  directories.
- **Skill success fast path** — after a successful `smaller prepare ... --json`, the Skill treats
  the Core JSON and its exact `use_path` as authoritative and avoids redundant post-success
  diagnostics or verification probes.
- **Skill/docs distinguish config from managed-store output** — the macOS config location
  (`~/Library/Application Support/SmallerPlease/config.json`) is now documented distinctly from the
  managed-store derivative/cache locations, so agents do not confuse configuration with output.

### Known limitations / Planned

- Still **Planned**: automatic updates, the Chrome Web Store listing, and a public Media Pack
  download service. Intel (`x86_64`) and Windows are deferred; the Homebrew formula refuses Intel.

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
