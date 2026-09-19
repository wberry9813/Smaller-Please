# Install Smaller Please on macOS

Smaller Please is macOS-first. There is no server, no account, and no telemetry. Pick a path:

| Path | Who it is for | Status |
|---|---|---|
| [Installer app (recommended)](../DOWNLOAD.md) | normal users | **Implemented** (the Beta.3 DMG is signed with a Developer ID and notarized/stapled) |
| [Manual install / guided steps](manual-macos.md) | users who want to run each step themselves | **Implemented** |
| [Homebrew](homebrew.md) | technical users | **Planned** (no public tap today) |

The available paths reuse the same Core (`smaller`, legacy alias `contextslim`) and the same
`setup`/`doctor`/`repair` contract. Chrome's **Developer mode → Load unpacked** step cannot be
automated and is always done by you.

## What an install produces

| Component | What it is | Managed by |
|---|---|---|
| Smaller Please Core | the `smaller` CLI / native host binary (legacy alias `contextslim`) | the installer, or your `PATH` |
| Media Engine | a paired `ffmpeg` + `ffprobe` | an existing PATH/Homebrew/system backend, or the optional local **Media Pack** (`smaller media install`) |
| Native Host | `~/.contextslim-bridge/native/contextslim-native-host` + Chrome host manifest `com.contextslim.bridge.json` | `smaller setup` / `smaller native install` / `smaller repair` |
| Chrome extension | production bundle staged to the visible `~/Applications/Smaller Please Extension` (ownership marker + state stay in `~/Library/Application Support/SmallerPlease/extension/`) | Core stages it (`smaller setup` / `smaller extension install`); you load it at `chrome://extensions` |
| User data | `config.json`, managed store/cache, browser-local history/metrics | never deleted by default |

## Installer app (recommended)

Opening the downloaded `Smaller-Please-Installer-0.1.0-beta.3-macos-arm64.dmg` and clicking
**Install** does everything below user-level (no `sudo`, no `PATH` changes) into
`~/Library/Application Support/SmallerPlease/`:

1. verifies the embedded payload (Core, extension bundle, LGPL Media Pack) by pinned SHA-256 +
   version/architecture, then atomically installs its own managed Core to `bin/`;
2. runs the installed `smaller setup` (config, managed store, Native Host) and
   `smaller media install --source <embedded>` only when no compatible backend exists;
3. stages the extension to the visible `~/Applications/Smaller Please Extension` path and offers
   **Open Chrome Extensions** for the one-time Developer-mode / Load-unpacked step.

An existing Homebrew/system `smaller` or FFmpeg is never deleted, overwritten, or modified. The
Beta.3 DMG is **signed and notarized**; see [`../DOWNLOAD.md`](../DOWNLOAD.md) for the detailed
walkthrough.

## Requirements

- macOS 12+ on Apple Silicon (arm64).
- A paired `ffmpeg` + `ffprobe` on `PATH` (for example `brew install ffmpeg`), or the optional
  LGPL **Media Pack** installed from a local payload (`smaller media install --source <pack>`) —
  the installer handles this for you when no compatible backend exists.
- Google Chrome (or a Chromium browser that supports the Native Messaging host).

`/usr/bin/sips` is used opportunistically for HEIC images on macOS and is not a required
cross-platform dependency.

## Verify an install

Implemented commands:

```bash
smaller --version
smaller setup              # idempotent; config, store, Media Engine, Native Host, extension staging
smaller doctor --json      # read-only health report (six checks; exits 0)
smaller repair             # allowlisted fixes for native/config/store/extension drift
smaller extension status --json
smaller extension path --json   # the directory to pick in Chrome's Load unpacked
smaller native status --json
smaller config show --json
smaller media status --json     # optional LGPL Media Pack (installed/missing)
smaller uninstall --dry-run --json   # read-only plan for program/integration teardown
```

Then open the extension popup. The status line must read `Core 0.1.0 ready` (the version
comes from the handshake, not a hardcoded string in the extension).

## Uninstall

`smaller uninstall` removes program/integration files (installer-managed Core, Native Host,
staged extension, managed Media Pack) and keeps config, store/cache, browser-local data,
Homebrew, and user media. `--incomplete` cleans only owned interrupted-install remnants.
`--remove-settings` / `--remove-cache` are explicit opt-ins, and `--dry-run` mutates nothing.
See [`../uninstall.md`](../uninstall.md).

## Not implemented yet

- Top-level `smaller update` and automatic update distribution.
- A **public** Homebrew tap (no public tap exists today).
- A **public** Media Pack download service and Media Pack signing/notarization, plus Intel
  (`x86_64`) and Windows builds. The local distributable Media Pack itself **is** Implemented.
- Loading the unpacked extension into Chrome (Developer mode / Load unpacked stays manual;
  Core stages the bundle but never automates that Chrome step).

Do not document or script these as if they exist.
