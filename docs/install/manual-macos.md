# Manual macOS install (guided steps)

This guide walks from a Mac with no Smaller Please installation to a working Core + Media
Engine + Native Host + unpacked extension, ending with a green `doctor` and a successful
handshake. It is the same work the installer app performs, exposed as individual steps you can
run yourself.

The quick path is the installer app in [`../DOWNLOAD.md`](../DOWNLOAD.md). If you prefer to see
and control each step, continue below.

Core → Media Engine → config → Native Host → extension staging → Load unpacked → handshake →
doctor.

`smaller setup` runs the middle steps for you:

```bash
smaller setup       # config + Media Engine + Native Host + extension staging
```

`smaller setup --yes` runs non-interactively and never installs Homebrew. `smaller doctor`
reports health at any time, and `smaller repair` fixes allowlisted registration/config/store/
extension problems. Chrome's **Load unpacked** step (step 6) is always manual.

No step uses `sudo`. No step modifies a source file.

## 0. Prerequisites

- macOS 12+ on Apple Silicon (arm64).
- Google Chrome.
- A paired `ffmpeg` + `ffprobe` (step 2). Homebrew is the easiest source but is not required:
  any `ffmpeg`/`ffprobe` pair on `PATH` in the same directory works, or the optional local
  Media Pack.

Download and install the app first if you have not: [`../DOWNLOAD.md`](../DOWNLOAD.md).

## 1. Install Core

Run the installer from the DMG (see [`../DOWNLOAD.md`](../DOWNLOAD.md)) and click **Install**.
The installer places a managed Core under
`~/Library/Application Support/SmallerPlease/bin/`. Verify it resolves:

```bash
smaller --version
```

If `smaller` is not on your `PATH`, the installer also exposes the managed binary at the
location above; `smaller doctor` will report the core check.

### One-command path (recommended)

```bash
smaller setup
smaller doctor
```

`setup` is idempotent and reuses the same Core primitives `doctor`/`repair` use: it ensures
config and the managed store, resolves the Media Engine (offering Homebrew only on an
interactive TTY), registers the Native Host, **stages the production extension bundle** when
one is resolvable, and prints the manual Chrome steps. It never loads the extension. When it
reports `Overall: OK` (or only the browser-extension user action remains), continue at
step 6. The step-by-step sections below are the equivalent manual path.

## 2. Media Engine

Install a paired backend:

```bash
brew install ffmpeg
```

If you do not use Homebrew, provide `ffmpeg` and `ffprobe` on `PATH` from any source, or install
the optional local LGPL Media Pack from a payload you already have.

Verify Core sees them:

```bash
smaller doctor --json
```

Expected: the `media_engine` check is `ok` with a `capabilities` map (probe, image, H.264,
AAC, MP4/MOV, scale, yuv420, metadata, faststart) all `true`, and `sips.status` is `ok` on
macOS. A present-but-broken backend reports `media_capability_missing`.

> `smaller setup` may offer `brew install ffmpeg` on an interactive TTY only;
> `--yes`/`--json`/non-TTY never prompt and never install Homebrew. When no compatible backend
> exists, the optional **Media Pack** (LGPL-only, macOS arm64) can be installed locally with
> `smaller media install --source <pack>` or accepted interactively; it is never downloaded
> automatically. See [`../troubleshooting/media-engine.md`](../troubleshooting/media-engine.md).

## 3. Config (optional)

Core config lives at `~/Library/Application Support/SmallerPlease/config.json`. An existing
legacy `ContextSlim/config.json` is migrated there automatically on first load (the legacy
file is left untouched). Defaults are fine for most users. Inspect or change it with:

```bash
smaller config show --json
smaller config set store-dir ~/SmallerPleaseStore
smaller config get store-dir
```

Precedence is **explicit CLI flag > persisted config > built-in default**. Changing
`store-dir` does not move old content; clean the previous store separately.

## 4. Native Host

Register the Chrome Native Messaging host:

```bash
smaller native install
smaller native status
```

`native install` copies the running binary to
`~/.contextslim-bridge/native/contextslim-native-host` and writes
`~/Library/Application Support/Google/Chrome/NativeMessagingHosts/com.contextslim.bridge.json`.
It is idempotent, writes nothing outside `$HOME`, needs no `sudo`, and never touches other
hosts.

`native status` must show `launcher current: true`. If registration is missing or the launcher
is stale, `smaller repair` reinstalls it and re-runs the handshake. See
[`../troubleshooting/native-host.md`](../troubleshooting/native-host.md).

## 5. Stage the extension into its fixed directory

`smaller setup` already does this. To stage/refresh explicitly:

```bash
smaller extension install
smaller extension status --json
```

Installing writes the verified bundle to the visible runtime root, with ownership metadata kept
separate:

```
~/Applications/Smaller Please Extension/     # Load-unpacked target (contains manifest.json)
  manifest.json  _locales/  dist/  icons/  popup/  install/

~/Library/Application Support/SmallerPlease/extension/   # metadata (hidden)
  .smaller-extension.json     # ownership marker + runtime-path binding
  state.json                  # lifecycle state
```

`smaller extension path` prints the visible `~/Applications/Smaller Please Extension` directory
to pick in Chrome. Staging never downloads anything, never modifies a source file, and leaves the
previous install intact if a bundle is rejected. `smaller setup`/`smaller repair` use the same
primitive.

## 6. Load the unpacked extension in Chrome (manual)

Chrome's security model does not allow a program to do this for you, so it is always a manual
step. Do not point any tooling at your real Chrome profile.

In Chrome:

1. Open `chrome://extensions`.
2. Enable **Developer mode** (top right).
3. Click **Load unpacked**.
4. Select `~/Applications/Smaller Please Extension`
   (`smaller extension path` prints the exact path).

Core can also reveal the directory: `smaller extension open` (macOS) reveals it in Finder and
opens `chrome://extensions`; it never enables Developer mode or clicks Load unpacked.

## 7. Handshake

Open the Smaller Please popup (click the extension icon) or the install page. The extension runs
`cs:handshake` through its service worker to the Native Host. A correct install shows:

```
Core 0.1.0 ready
```

If it shows `Core settings unavailable` or an install-instructions page, see
[`../troubleshooting/native-host.md`](../troubleshooting/native-host.md).

## 8. Final check

```bash
smaller doctor --json
smaller extension status --json
smaller native status --json
smaller config show --json
smaller clean --dry-run --json
```

`smaller doctor` is the read-only health report; it exits `0`. If the Native Host or config /
store checks are not `ok`, `smaller repair --dry-run` prints the plan and `smaller repair`
applies the allowlisted fixes. Then drag a large image into ChatGPT or Claude and confirm the
in-page chip reports the saved bytes. The source file is never modified.

## What is not part of this install

Planned only, not available: top-level `smaller update`, automatic updates, a public
Homebrew tap, and a public Media Pack download/signing service.
Implemented: `smaller setup`, `smaller doctor`, `smaller repair`,
`smaller extension install|status|path|open|uninstall` (fixed-directory staging; Chrome's
Developer mode / Load unpacked stays manual), `smaller uninstall` (program/integration teardown
with `--incomplete` cleanup and `--remove-settings`/`--remove-cache` opt-ins), and the local
distributable Media Pack (`smaller media install|status|path|uninstall`).
