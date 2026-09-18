# CLI reference

Technical guide for **Smaller, Please Core** — the `smaller` command, its configuration, and
the install/health lifecycle.

New to the product? Start with the [product README](../README.md) and
[`DOWNLOAD.md`](DOWNLOAD.md) instead.

Related documentation:

- [`install/macos.md`](install/macos.md) and [`install/homebrew.md`](install/homebrew.md) —
  install paths.
- [`uninstall.md`](uninstall.md) — the uninstall lifecycle.
- [`troubleshooting/`](troubleshooting/core.md) — diagnosing Core problems.
- [`EXTENSION_SETTINGS.md`](EXTENSION_SETTINGS.md) — the browser extension's own settings.

## Requirements

- macOS (first supported platform).
- An `ffmpeg` / `ffprobe` pair on `PATH`, **or** the optional LGPL Media Pack on Apple Silicon.

On Apple Silicon the optional LGPL **Media Pack** provides a paired `ffmpeg`/`ffprobe` when no
compatible backend exists:

```bash
smaller media status --json
smaller media install --source /path/to/smaller-media-7.1-lgpl.1-macos-arm64.tar.gz
smaller media uninstall --yes
```

`/usr/bin/sips` is used opportunistically for HEIC on macOS; it is not a required cross-platform
dependency.

## Install and health

Core owns one shared install/health flow (all **Implemented**):

```bash
smaller setup            # config + managed store + Media Engine + Native Host + extension staging
smaller doctor           # read-only health report; exits 0
smaller repair           # allowlisted fixes for native/config/store/extension drift
smaller extension status --json
smaller extension path --json   # the directory to pick in Chrome's Load unpacked
smaller media status --json     # optional LGPL Media Pack (installed/missing)
smaller uninstall --dry-run --json   # read-only plan for program/integration teardown
```

- `setup` is idempotent. `--yes` runs non-interactively and never installs Homebrew;
  `--json` never prompts. It stages a valid production bundle automatically when one is
  resolvable.
- `smaller extension install|status|path|open|uninstall` manage the unpacked bundle at
  `~/Applications/Smaller Please Extension` (the ownership marker and lifecycle state stay in
  `~/Library/Application Support/SmallerPlease/extension/`). Core stages/updates it atomically;
  Chrome's **Developer mode → Load unpacked** step is still a one-time manual action
  (`smaller extension open` reveals the directory and opens `chrome://extensions`). Extension
  **staged ≠ loaded in Chrome ≠ connected**; `load_state` is intentionally `unknown`.
- `doctor` reports six checks (`core`, `config`, `storage`, `media_engine`, `native_host`,
  `browser_integration`) with stable status/code/detail fields. `doctor` never migrates config,
  adopts a store, or write-probes a directory. See [`troubleshooting/`](troubleshooting/core.md).
- `smaller uninstall [--incomplete] [--dry-run] [--yes] [--json] [--remove-settings]
  [--remove-cache]` removes program/integration files and keeps config, store/cache,
  browser-local data, Homebrew, and user media. `--incomplete` is the installer's
  **Clean Incomplete Installation**; `--remove-settings`/`--remove-cache` are explicit opt-ins,
  and `--dry-run` mutates nothing. The Chrome **Remove** click stays manual. See
  [`uninstall.md`](uninstall.md).
- Homebrew: a **public** `homebrew-smaller-please` tap is **Planned** and is not available yet.
  See [`install/homebrew.md`](install/homebrew.md).

Planned, not available: a public Homebrew tap, a **public** Media Pack download/signing service,
the Chrome Web Store listing, and top-level `smaller update`. The local distributable **Media
Pack** (LGPL-only FFmpeg pair, macOS arm64) is **Implemented** as a separate artifact via
`smaller media`; Core never downloads it automatically. See
[`THIRD_PARTY_NOTICES.md`](../THIRD_PARTY_NOTICES.md).

## Usage

```bash
smaller image ./screenshot.png
smaller image ./photo.heic --max-edge 2048 --quality 80
smaller image ./screenshot.png --dry-run
smaller image ./screenshot.png --json
smaller batch ./screenshots
smaller batch ./screenshots --output /tmp/ai-ready --json
```

The legacy command `contextslim` is still installed and works as a compatibility alias (it
prints a one-line notice to stderr; JSON stdout stays pure). New scripts and docs use
`smaller`.

Default behavior:

- source is never changed
- images below 2 MiB and videos below 10 MiB are skipped by default; the built-in defaults,
  the persisted config, and the explicit flags combine as **explicit flag > persisted config
  > built-in default**
- `--image-min-size-mb` / `--video-min-size-mb` set the minimum in MiB (`0` always attempts),
  and a media-specific flag wins over the shared `--min-size-mb` fallback
- longest edge is capped at 2048 px
- HEIC/JPEG/TIFF become JPEG derivatives
- PNG stays PNG in this MVP to avoid destroying UI/transparency unexpectedly
- output defaults to `.contextslim/` beside the source
- metadata is stripped by the ffmpeg backend
- if the result is not smaller, it is deleted and the source is kept

## Configuration

**Core/CLI is the source of truth for all non-browser-specific settings.** The browser
extension is a UI frontend over the same config and never keeps a second copy of it.

The config lives at `~/Library/Application Support/SmallerPlease/config.json` on macOS
(`$XDG_CONFIG_HOME/smaller-please/config.json` or `~/.config/smaller-please/config.json`
elsewhere). `SMALLER_CONFIG_DIR` overrides the directory for portable setups; the legacy
`CONTEXTSLIM_CONFIG_DIR` is still honored. On first load, an existing legacy
`ContextSlim/config.json` is copied to the canonical path atomically (the legacy file is
left untouched).

```json
{
  "schema_version": 1,
  "image_min_bytes": 2097152,
  "video_min_bytes": 10485760,
  "min_savings_percent": 10,
  "bridge_store_dir": "~/.contextslim-bridge/store"
}
```

```bash
smaller config show [--json]
smaller config get image-min-size-mb
smaller config set image-min-size-mb 5
smaller config set video-min-size-mb 20
smaller config set min-savings-percent 12.5
smaller config set store-dir ~/SmallerPleaseStore
smaller config reset min-savings-percent
smaller config reset --all
```

The file is written atomically, unknown future fields are preserved, and a corrupt file
recovers to defaults with a stderr diagnostic.

### Core-owned vs extension-owned

| Setting | Owner |
|---|---|
| Image/video minimum size, minimum savings, store location | Core config (CLI + popup) |
| History limit, popup UI state, chip/notification behavior | Browser extension (`chrome.storage.local`) |
| Metrics/activity history | Browser extension (local only) |

The legacy browser-only threshold store is imported into Core once; after that the popup
reads/writes Core, so a Terminal `smaller config set` is reflected on the next popup
open.

### Clean

```bash
smaller clean [store|cache|inbox|all] [--dry-run] [--yes] [--json]
```

`clean` reports the same byte counts as the popup's storage usage and only ever removes
marker-validated Smaller, Please-managed content plus the bridge-owned inbox. `--dry-run`
deletes nothing, and a non-interactive run refuses without `--yes`. Source files, the
config file, and browser-local history/metrics are never touched. Changing the store
location does not migrate old content; clean the previous store separately. See
[`troubleshooting/storage.md`](troubleshooting/storage.md).

## Agent contract

Agents should use the optimized derivative as visual input, never overwrite the source, and fall
back to the source if Smaller, Please reports `skipped` or fails.

JSON output is intended for Skills and agent wrappers.

## Scope

Implemented:

- single image
- recursive batch
- dry-run
- JSON output
- byte savings statistics
- ffmpeg backend
- macOS HEIC `sips` backend
- never-grow guard
- `setup` / `doctor` / `repair`, the extension/native/media lifecycle, and `uninstall`

Planned:

- context budget allocation across batches
- a public Homebrew tap and public distribution
- a public Media Pack download + signing/notarization service
- automatic updates, a Chrome Web Store listing, Intel (`x86_64`), and Windows
