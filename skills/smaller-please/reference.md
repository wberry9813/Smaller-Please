# Smaller, Please — verified command reference

All commands below are captured from the shipped `smaller` CLI. Do not invent flags.

## `smaller optimize [OPTIONS] <INPUT>`

Optimize an image, a video, or a directory (a directory is recursively optimized). Sources are
never modified.

Options:

- `--mode <smart|maximum>` — default `smart` (the historic pipeline; Free). `maximum` compresses
  harder and requires the `compression.maximum` capability (Pro). Under Free it is denied with the
  stable `CAPABILITY_REQUIRED` code and nothing is written.
- `--metadata <remove-all>` — the public value is `remove-all` (remove all
  removable metadata, requires the `metadata.control` capability; denied with `CAPABILITY_REQUIRED`
  under Free). With no `--metadata`, Core does **not intentionally strip** metadata: it omits
  explicit removal arguments and leaves handling to the selected backend (some re-encoding
  backends, including the current FFmpeg JPEG path, may naturally omit source EXIF/XMP/IPTC); only
  `remove-all` is an explicit stripping request. `keep`/`remove-privacy` are internal/reserved and
  are not advertised or sellable values.
- `--max-edge` — default `2048`.
- `--quality`
- `--crf`
- `--min-savings`
- `--output`
- `--force`
- `--dry-run`
- `--json`
- `--image-min-size-mb`
- `--video-min-size-mb`
- `--min-size-mb`

## `smaller prepare [OPTIONS] <PATH>`

Prepare one image or video for agent input; emits a frozen JSON plan with `use_path`.

Options include `--max-edge`, `--quality`, `--crf`, `--min-savings`, `--output`, `--json`,
`--image-min-size-mb`, `--video-min-size-mb`, `--min-size-mb`.

## `smaller prepare-batch [OPTIONS] <PATHS>...`

Prepare several files or directories in one JSON document. Same options as `prepare`.

## `smaller doctor [--json]`

Show backend availability.

## `smaller license status [--json]`

Show the local license status (plan, granted capabilities, expiration).

## `smaller license activate <KEY> [--json]`

Verify and store a license key locally (no network). An invalid key writes nothing.

## `smaller license remove [--json]`

Delete the local license file and return to Free (idempotent).

The Beta.6 release build trusts the Production key; a valid Production-signed
licence activates Pro. Without a valid licence, the gated flags (`--mode maximum`, `--metadata remove-all`)
are denied with `CAPABILITY_REQUIRED` and nothing is written. The local licence workflow is
`smaller license activate <KEY>` / `status` / `remove` and is fully offline (no network).

## Other top-level commands

- `setup [--json] [--yes]` — set up Core: configuration, managed store, media engine, and native host.
- `repair [--dry-run] [--yes] [--json]` — repair allowlisted installation problems.
- `config show|get|set|reset` — read or update the persisted Core configuration.
- `clean [store|cache|inbox|all] [--dry-run] [--yes] [--json]` — report or remove Smaller, Please-managed cached media.
- `native install|status|uninstall|host` — manage the Chrome Native Messaging host for the browser extension.
- `extension install|status|path|open|uninstall` — stage, inspect, or remove the unpacked Chrome extension bundle.
- `media status|install|path|uninstall` — install, inspect, or remove the optional LGPL Media Pack.
- `uninstall [--incomplete] [--dry-run] [--yes] [--json] [--remove-settings] [--remove-cache]` — remove program/integration files, or clean owned interrupted-install remnants.
