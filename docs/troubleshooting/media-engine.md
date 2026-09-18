# Troubleshooting: Media Engine (`ffmpeg` / `ffprobe`)

Core does not implement codecs; it drives a paired `ffmpeg` + `ffprobe`. On macOS,
`/usr/bin/sips` is used opportunistically for HEIC.

## Diagnose

```bash
smaller doctor --json      # media_engine check: source/dir/versions + capabilities map
smaller setup              # resolve + capability test; interactive Homebrew/Media Pack offer
smaller media status --json  # optional LGPL Media Pack (installed/missing)
smaller media path --json    # the active pack bin/ (when installed)
which ffmpeg ffprobe
ffmpeg -version
ffprobe -version
```

The `media_engine` check reports one of:

- `media_engine_ok` — a paired backend passed the real capability test.
- `media_backend_missing` — no paired `ffmpeg`/`ffprobe` found.
- `media_capability_missing` — a backend was found but is missing a required capability; the
  `capabilities` map names which (`probe`, `image`, `video`, `h264`, `aac`, `mp4`, `mov`,
  `scale`, `yuv420`, `metadata`, `faststart`).

`media_engine.details.source` is one of `custom` / `homebrew` / `system` / `bundled_media_pack`
(plus `unknown`/`unavailable`), and `video_encoder` is `libx264` for an x264 backend or
`h264_videotoolbox` for the Media Pack. For `bundled_media_pack` the details also carry
`media_pack_version` and `license_profile` (`lgpl`).

The legacy `ffmpeg`/`ffprobe` keys still report `ok` / `not_found` / `failed_to_launch`.
`.contextslim` generated output is never scanned for media; that is unrelated to backend
availability.

## Fix: install or expose a backend

```bash
brew install ffmpeg     # or: smaller setup (offers this on an interactive TTY only)
```

Homebrew is one supported source; any `ffmpeg`/`ffprobe` pair on `PATH` works. Core's native
host discovers the pair from the inherited `PATH` plus `/opt/homebrew/bin`, `/usr/local/bin`,
`/opt/local/bin`, `/usr/bin`, and `/bin`; **both** tools must exist and be executable in the
same directory. The host logs a non-sensitive line to stderr:

```
contextslim-native-host: backends ffmpeg=found@<dir> ffprobe=found@<dir> path_augmented=true
```

`found@<dir>` names only the backend directory, never a media path.

## Browser: "Media engine not found."

The in-page chip / popup maps the stable code `backend_not_found` to localized copy. The
extension cannot see your shell `PATH`; Chrome launches the host with a minimal GUI `PATH`, so
the host augments it during startup. If it still fails:

1. Confirm `smaller doctor` is `ok/ok` in your terminal.
2. Re-run `smaller native install` (the host binary is the one that discovers backends).
3. Reload the extension and hard-refresh the site.

## Behavior when the backend is missing

Media is never lost. `prepare` and the browser fall back to the original (`use_path` points
at the source); CLI `image`/`video`/`batch` report an error rather than corrupting anything.
Sources are never modified.

## Homebrew offer (Implemented, interactive only)

When no capable backend exists and Homebrew is present, `smaller setup` may print exactly:

```
FFmpeg is required for media optimization. Install FFmpeg using Homebrew? [y/N]
```

Only an explicit `y` runs `brew install ffmpeg` (argv only, progress inherited), and Core
re-runs the capability test afterwards. `--yes`, `--json`, and non-TTY stdin never prompt and
never install. Core never installs Homebrew itself and never downloads an executable.

## Media Pack (Implemented; local distributable)

The LGPL-only **Media Pack** (`ffmpeg`/`ffprobe`, macOS arm64, `h264_videotoolbox` + native
AAC) is the fallback when no compatible PATH/Homebrew/system backend exists. It is a
**separate artifact** and is never downloaded automatically.

```bash
smaller media status --json
smaller media install --source /path/to/smaller-media-7.1-lgpl.1-macos-arm64.tar.gz
smaller media path --json
smaller media uninstall --yes
```

`smaller setup` offers it only with explicit consent when no backend was found (after the
interactive Homebrew path); `--yes`/`--json`/non-TTY never download and `--yes` may only install
an already-present local payload. `setup`/`repair` never install Homebrew and never download a
Media Pack. `doctor` reports `source: bundled_media_pack` and always runs the real capability
test — never trusting the manifest alone. `repair` can recover a managed pack marker/pointer but
never downloads.

A Homebrew/system/custom backend always wins over an installed Media Pack, so a healthy
Homebrew environment is unchanged. See
[`../../THIRD_PARTY_NOTICES.md`](../../THIRD_PARTY_NOTICES.md) for the licensing and attribution
of the Media Pack.
